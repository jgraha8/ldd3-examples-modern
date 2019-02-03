<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. LDD3 Examples Kernel 4.x Support</a>
<ul>
<li><a href="#sec-1-1">1.1. struct file\<sub>operations</sub> -&gt; ioctl removed</a>
<ul>
<li><a href="#sec-1-1-1">1.1.1. See: </a></li>
<li><a href="#sec-1-1-2">1.1.2. See the patch for the ioctl rework: </a></li>
</ul>
</li>
<li><a href="#sec-1-2">1.2. Getting the current process UID, EUID, etc.</a>
<ul>
<li><a href="#sec-1-2-1">1.2.1. The task struct was updated to use the cred struct in commit:</a></li>
<li><a href="#sec-1-2-2">1.2.2. Getting the UID, EUID, etc.</a></li>
<li><a href="#sec-1-2-3">1.2.3. The UID, EUID, etc. are stored as kuid<sub>t</sub> types.</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>



# LDD3 Examples Kernel 4.x Support<a id="sec-1" name="sec-1"></a>

Notes for the scull module updates (examples/scull)

## struct file\\<sub>operations</sub> -> ioctl removed<a id="sec-1-1" name="sec-1-1"></a>

When the Big Kernel Lock (BKL) was removed (fully removed in 2.6.39)
ioctl went with it. One must use unlocked<sub>ioctl</sub> when setting the
file operations struct.

See: <https://kernelnewbies.org/BigKernelLock>

### See: <http://derekmolloy.ie/writing-a-linux-kernel-module-part-2-a-character-device/><a id="sec-1-1-1" name="sec-1-1-1"></a>

Provides an example including the addition of adding locks with
the protecting access to a device. Their example locks on open()
and unlocks on release().

### See the patch for the ioctl rework: <https://lwn.net/Articles/119656/><a id="sec-1-1-2" name="sec-1-1-2"></a>

The ioctl -> unlocked<sub>ioctl</sub> signature change:

int (\*ioctl) (struct inode \*, struct file \*, unsigned int, unsigned long);
long (\*unlocked<sub>ioctl</sub>) (struct file \*filp, unsigned int, unsigned long);

The inode is obtained from \`\`filp->f<sub>dentry</sub>->d<sub>inode\`\`</sub>.

## Getting the current process UID, EUID, etc.<a id="sec-1-2" name="sec-1-2"></a>

### The task struct was updated to use the cred struct in commit:<a id="sec-1-2-1" name="sec-1-2-1"></a>

See: <https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=b6dff3ec5e116e3af6f537d4caedcad6b9e5082a>

### Getting the UID, EUID, etc.<a id="sec-1-2-2" name="sec-1-2-2"></a>

See: <https://stackoverflow.com/questions/39229639/how-to-get-current-processs-uid-and-euid-in-linux-kernel-4-2>

When you have the task struct, the following macros should be used (from commit b6dff3ec):

\#define task<sub>uid</sub>(task)    ((task)->cred->uid)
\\#define task<sub>gid</sub>(task)    ((task)->cred->gid)
\\#define task<sub>euid</sub>(task)   ((task)->cred->euid)
\\#define task<sub>egid</sub>(task)   ((task)->cred->egid)

### The UID, EUID, etc. are stored as kuid<sub>t</sub> types.<a id="sec-1-2-3" name="sec-1-2-3"></a>

For comparing \`\`kuid<sub>t\`\`</sub>, we use \`\`uid<sub>eq</sub>()\`\`, and related macros from \`\`<linux/uidgid.h>\`\`.