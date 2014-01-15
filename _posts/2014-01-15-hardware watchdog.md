---
layout: post
category : technical
tagline: "hardware watchdog"
tags : [BMC, Linux]
---
{% include JB/setup %}


之前听说过Linux上有个char设备/dev/watchdog，并且自己试验过，但是一直不知道内部是怎么实现的。今天看到了一个bug，于是仔细看了一下。

其实计时功能是由BMC提供的，这相当于在os之外还有一个系统在监控os。ipmi的driver会注册这个设备，kernel或者user space的process通过周期性地写入字符来保证timer不会timeout。写操作经过vfs层，最后其实就是变成了reset timer的操作，最后driver会通过kcs interface把信息通过某个总线给BMC（在我们某些系统里是lpc，取决于主板怎么设计了）。


实现在 linux-x.x.xx/drivers/char/ipmi/ipmi_watchdog.c

misc设备，本质上就是主设备号为10的char设备。watchdog设备的minor#是130.



pet watchdog，可以看到，如果nowayout是0的话，那么可以写入V来停止watchdog。
{% highlight c %}
static ssize_t ipmi_write(struct file *file,
              const char  __user *buf,
              size_t      len,
              loff_t      *ppos)
{
    int rv;

    if (len) {
            if (!nowayout) {
                size_t i;

            /* In case it was set long ago */
            expect_close = 0;

                for (i = 0; i != len; i++) {
                char c;

                if (get_user(c, buf + i))
                    return -EFAULT;
                if (c == 'V')
                    expect_close = 42;
            }
        }
        rv = ipmi_heartbeat();
        if (rv)
            return rv;
        return 1;
    }
    return 0;
}
{% endhighlight %}

当watchdog timeout是会发生什么？ 这决定于一些参数的配置，可以reset，可以poweroff，而且还可以配置在timeout之前的某个时间也触发一些动作（触发nmi，或者是正常的中断），这些都是可以通过sysfs配置的 （`/sys/module/ipmi_watchdog/parameters`）。

在我们的系统上，`pretimeout`配置成了在`timeout`之前255秒触发一个中断，而且`isr`实现成了进行`crashdump`。这么长时间其实就是为了能够把`crashdump`存到磁盘上，因为系统的内存可是上百G。最后BMC会写两条SEL，等系统起来以后用`ipmitool sel list`就可以看到。


{% highlight c %}
static void ipmi_wdog_pretimeout_handler(void *handler_data)
{
    if (preaction_val != WDOG_PRETIMEOUT_NONE) {
        if (preop_val == WDOG_PREOP_PANIC) {
            if (atomic_inc_and_test(&preop_panic_excl))
                panic("Watchdog pre-timeout");
        } else if (preop_val == WDOG_PREOP_GIVE_DATA) {
            spin_lock(&ipmi_read_lock);
            data_to_read = 1;
            wake_up_interruptible(&read_q);
            kill_fasync(&fasync_q, SIGIO, POLL_IN);

            spin_unlock(&ipmi_read_lock);
        }
    }

    /* On some machines, the heartbeat will give
       an error and not work unless we re-enable
       the timer.   So do so. */
    pretimeout_since_last_heartbeat = 1;
}
{% endhighlight %}


ipmi driver听起来简单，看起来还是挺琐碎的，有空再看看。
