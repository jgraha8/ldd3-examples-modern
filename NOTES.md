# LDD3 Examples Kernel 4.x Support

Notes for the port of the Linux Device Drivers 3rd Edition examples to the 4.x kernel series.


## `struct file_operations` -> `ioctl` removed

When the Big Kernel Lock (BKL) was removed (fully removed in 2.6.39)
ioctl went with it. One must use `unlocked_ioctl` when setting the
file operations struct.

See: <https://kernelnewbies.org/BigKernelLock>


### See: <http://derekmolloy.ie/writing-a-linux-kernel-module-part-2-a-character-device/>

Provides an example including the addition of adding locks with the
protecting access to a device. Their example locks on open() and
unlocks on release().


### See the patch for the ioctl rework: <https://lwn.net/Articles/119656/>

The `ioctl` -> `unlocked_ioctl` signature change:

    int (*ioctl) (struct inode *, struct file *, unsigned int, unsigned long);

    long (*unlocked_ioctl) (struct file *filp, unsigned int, unsigned long);

The inode is obtained from `filp->f_dentry->d_inode` in the `unlocked_ioctl` procedure.


## Getting the current process UID, EUID, etc.


### The task struct was updated to use the cred struct in commit:

See: <https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=b6dff3ec5e116e3af6f537d4caedcad6b9e5082a>


### Getting the UID, EUID, etc.<a id="sec-1-2-2" name="sec-1-2-2"></a>

See: <https://stackoverflow.com/questions/39229639/how-to-get-current-processs-uid-and-euid-in-linux-kernel-4-2>

When you have the task struct, the following macros should be used (from commit [b6dff3ec](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=b6dff3ec5e116e3af6f537d4caedcad6b9e5082a)):

    #define task_uid(task)    ((task)->cred->uid)
    #define task_gid(task)    ((task)->cred->gid)
    #define task_euid(task)   ((task)->cred->euid)
    #define task_egid(task)   ((task)->cred->egid)


### The UID, EUID, etc. are stored as kuid_t types.

For comparing `kuid_t`, we use `uid_eq()`, and related macros from `<linux/uidgid.h>`.
