---
title: Android7.0init进程分析
date: 2018-04-08 15:21:27
categories: Android
tags: 
- Android启动
---


## ueventd简述及相关背景介绍
1. 与Linux相同，Android中的应用程序通过设备驱动访问硬件设备。设备节点文件是设备驱动的逻辑文件，应用程序使用设备节点文件来访问驱动程序
2. 在Linux中，运行所需的设备节点文件被被预先定义在“/dev”目录下。应用程序无需经过其它步骤，通过预先定义的设备节点文件即可访问设备驱动程序。 
但根据Android的init进程的启动过程，Android根文件系统的映像中不存在“/dev”目录，该目录是init进程启动后动态创建的
3. init进程创建子进程ueventd，并将创建设备节点文件的工作托付给ueventd。

ueventd通过两种方式创建设备节点文件。 
- 第一种方式对应“冷插拔”（Cold Plug），即以预先定义的设备信息为基础，当ueventd启动后，统一创建设备节点文件。这一类设备节点文件也被称为静态节点文件。 
- 第二种方式对应“热插拔”（Hot Plug），即在系统运行中，当有设备插入USB端口时，ueventd就会接收到这一事件，为插入的设备动态创建设备节点文件。这一类设备节点文件也被称为动态节点文件。

从内核2.6x开始引入udev(userspace device)，udev以守护进程的形式运行。当设备驱动被加载时，它会获取主设备号、次设备号，以及设备类型，而后在“/dev”目录下自动创建设备节点文件。
热插拔时，设备连接后，内核调用驱动程序加载信息到/sys下，然后驱动程序发送uevent到udev； 
冷插拔时，udev主动读取/sys目录下的信息，然后触发uevent给自己处理。之所以要有冷插拔，是因为内核加载部分驱动程序信息的时间，早于启动udev的时间。
## ueventd的启动
在init解析init.rc文件中，我们可以看到在on early-init中

```
on early-init
    # Set init and its forked children's oom_adj.
    write /proc/1/oom_score_adj -1000

    # Disable sysrq from keyboard
    write /proc/sys/kernel/sysrq 0

    # Set the security context of /adb_keys if present.
    restorecon /adb_keys

    # Shouldn't be necessary, but sdcard won't start without it. http://b/22568628.
    mkdir /mnt 0775 root system

    # Set the security context of /postinstall if present.
    restorecon /postinstall

    start ueventd

```
分析init流程，可以知道此处start操作将启动ueventd服务,start操作对应system/core/init/builtins.cpp中的do_start()函数。

##  ueventd代码分析
 ueventd相关代码位于 system/core/init/ueventd.cpp，
 
```
int ueventd_main(int argc, char **argv)
{
    /*
     * init sets the umask to 077 for forked processes. We need to
     * create files with exact permissions, without modification by
     * the umask.
     */
    umask(000);

    /* Prevent fire-and-forget children from becoming zombies.
     * If we should need to wait() for some children in the future
     * (as opposed to none right now), double-forking here instead
     * of ignoring SIGCHLD may be the better solution.
     */
    signal(SIGCHLD, SIG_IGN);

    open_devnull_stdio();
    klog_init();
    klog_set_level(KLOG_NOTICE_LEVEL);

    NOTICE("ueventd started!\n");

    selinux_callback cb;
    cb.func_log = selinux_klog_callback;
    selinux_set_callback(SELINUX_CB_LOG, cb);

    std::string hardware = property_get("ro.hardware");

    ueventd_parse_config_file("/ueventd.rc");
    ueventd_parse_config_file(android::base::StringPrintf("/ueventd.%s.rc", hardware.c_str()).c_str());

    device_init();

    pollfd ufd;
    ufd.events = POLLIN;
    ufd.fd = get_device_fd();

    while (true) {
        ufd.revents = 0;
        int nr = poll(&ufd, 1, -1);
        if (nr <= 0) {
            continue;
        }
        if (ufd.revents & POLLIN) {
            handle_device_fd();
        }
    }

    return 0;
}

```
与之相关的代码还有devices.cpp和uevnetd_parser.cpp, device_init()函数代码如下
```
void device_init() {
    sehandle = selinux_android_file_context_handle();
    selinux_status_open(true);

    /* is 256K enough? udev uses 16MB! */
    device_fd = uevent_open_socket(256*1024, true);
    if (device_fd == -1) {
        return;
    }
    fcntl(device_fd, F_SETFL, O_NONBLOCK);

    if (access(COLDBOOT_DONE, F_OK) == 0) {
        NOTICE("Skipping coldboot, already done!\n");
        return;
    }

    Timer t;
    coldboot("/sys/class");
    coldboot("/sys/block");
    coldboot("/sys/devices");
    close(open(COLDBOOT_DONE, O_WRONLY|O_CREAT|O_CLOEXEC, 0000));
    NOTICE("Coldboot took %.2fs.\n", t.duration());
}

```
上述代码其实就是完成冷启动的处理。实际上，当冷启动完成之后，该函数创建了一个名为COLDBOOT_DONE的文件。在system/core/init/init.cpp中有这样一段代码

```
 am.QueueBuiltinAction(wait_for_coldboot_done_action, "wait_for_coldboot_done");
```
找到处理该Action的相关函数

```
static int wait_for_coldboot_done_action(const std::vector<std::string>& args) {
    Timer t;

    NOTICE("Waiting for %s...\n", COLDBOOT_DONE);
    // Any longer than 1s is an unreasonable length of time to delay booting.
    // If you're hitting this timeout, check that you didn't make your
    // sepolicy regular expressions too expensive (http://b/19899875).
    if (wait_for_file(COLDBOOT_DONE, 1)) {
        ERROR("Timed out waiting for %s\n", COLDBOOT_DONE);
    }

    NOTICE("Waiting for %s took %.2fs.\n", COLDBOOT_DONE, t.duration());
    return 0;
}

```
注意其中的注释。Android7.0的开发者认为，coldboot的过程不会大于1秒，这里的wait_for_file()函数实际上就是在1秒内检测COLDBOOT_DONE这个宏定义的文件是否存在，若存在，则可以认为coldboot过程已经结束，这里一旦1秒内没有检测到文件存在，只会报告一个错误，init启动过程仍然会继续，但最后面会出现被kill的情况。鉴于以上情况，我们采取的策略是增大wait_for_file函数的等待时间。

```
if (wait_for_file(COLDBOOT_DONE, 5)) {
        ERROR("Timed out waiting for %s\n", COLDBOOT_DONE);
    }
```


