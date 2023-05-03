Download Link: https://assignmentchef.com/product/solved-cs1550-project1-syscalls-for-synchronization
<br>



Anytime we share data between two or more processes or threads, we run the risk of having a race condition where our data could become corrupted. In order to avoid these situations, we have discussed various mechanisms to ensure that one program’s critical regions are guarded from another’s.

<a name="_Toc7475"></a>Project Details

<a name="_Toc7476"></a>Syscalls for Synchronization




We need to create a semaphore data type and the two operations we described in class, down() and up(). To encapsulate the semaphore, we’ll make a simple struct that contains an integer value and a priority queue of processes:

struct cs1550_sem

{

int value;

//Some priority queue of your devising

};




The priority queue stores the processes waiting on the semaphore and enables higher priority processes to be woken up first.




We will then make two new system calls with the following signatures:

asmlinkage long sys_cs1550_down(struct cs1550_sem *sem) asmlinkage long sys_cs1550_up(struct cs1550_sem *sem)

to operate on our semaphores.

<h2><a name="_Toc7477"></a>Priority</h2>

The priority of the process can be accessed using the following function. <em>Note that a higher value denotes higher priority.  </em>

int priority = task_nice(current);

<h2><a name="_Toc7478"></a>Sleeping</h2>




As part of your down() operation, there is a potential for the current process to sleep. In Linux, we can do that as part of a two-step process.

<ul>

 <li>Mark the task as not ready (but can be awoken by signals): set_current_state(TASK_INTERRUPTIBLE);</li>

 <li>Invoke the scheduler to pick a ready task: schedule();</li>

</ul>

<h2><a name="_Toc7479"></a>Waking Up</h2>




As part of up(), you potentially need to wake up a sleeping process. You can do this via:

wake_up_process(sleeping_task);

Where sleeping_task is a struct task_struct that represents a process put to sleep in your down(). You can get the current process’s task_struct by accessing the global variable current. You may need to save these someplace.

<h2><a name="_Toc7480"></a>Atomicity</h2>




We need to implement our semaphores as part of the kernel because we need to do our increment or decrement and the following check on it <strong>atomically</strong>. In class we said that we’d disable interrupts to achieve this. In Linux, this is no longer the preferred way of doing in kernel synchronization due to the fact that we might be running on a multicore or multiprocessor machine. Instead, we’ll use something somewhat surprising: <em>spin locks</em>.

We can create a spinlock with a provided macro:

DEFINE_SPINLOCK(sem_lock);

We can then surround our critical regions with the following:

spin_lock(&amp;sem_lock); spin_unlock(&amp;sem_lock);




For each, feel free to draw upon the text and handouts for this course as well as 449.

<h2><a name="_Toc7481"></a>Adding a New Syscall</h2>




To add a new syscall to the Linux kernel, there are three main files that need to be modified:

<ol>

 <li>linux-2.6.23.1/kernel/sys.c</li>

</ol>

This file contains the actual implementation of the system calls.

<ol start="2">

 <li>linux-2.6.23.1/arch/i386/kernel/syscall_table.S</li>

</ol>

This file declares the number that corresponds to the syscalls

<ol start="3">

 <li>linux-2.6.23.1/include/asm/unistd.h</li>

</ol>

This file exposes the syscall number to C programs which wish to use it.

<h2><a name="_Toc7482"></a>Testing the syscalls</h2>




As you implement your syscalls, you are also going to want to test them via a user-land program. The first thing we need is a way to use our new syscalls. We do this by using the syscall() function. The syscall function takes as its first parameter the number that represents which system call we would like to make. The remainder of the parameters are passed as the parameters to our syscall function. We have the syscall numbers exported as #defines of the form __NR_syscall via our unistd.h file that we modified when we added our syscalls.

We can write wrapper functions or macros to make the syscalls appear more natural in a C program. For example, you could write:

void down(struct cs1550_sem *sem) {

syscall(__NR_cs1550_down, sem);

}




<h1><a name="_Toc7483"></a>Hints</h1>




<h2><a name="_Toc7484"></a>Setting up the Kernel Source (To do in recitation)</h2>




<ol>

 <li>Copy the linux-2.6.23.1.tar.bz file to your local space under /u/OSLab/username cp /u/OSLab/original/linux-2.6.23.1.tar.bz2 .</li>

 <li>Extract</li>

</ol>

tar xfj linux-2.6.23.1.tar.bz2

<ol start="3">

 <li>Change into linux-2.6.23.1/ directory cd linux-2.6.23.1</li>

 <li>Copy the .config file cp /u/OSLab/original/.config .</li>

 <li>Build make ARCH=i386 bzImage</li>

</ol>

You should only need to do this once, however redoing step 2 will undo any changes you’ve made and give you a fresh copy of the kernel should things go horribly awry.




<h2><a name="_Toc7485"></a>Rebuilding the Kernel</h2>




To build any changes you made, from the linux-2.6.23.1/ directory, simply:

make ARCH=i386 bzImage

<h2><a name="_Toc7486"></a>QEMU Version</h2>

We will be using an x86-based version of Linux and QEMU for this project. The disk image and a copy of QEMU for windows are available on CourseWeb (qemu.zip). For Mac users, you can download an older but GUI-based application (Q.app) available on CourseWeb as well. Point it at the tty.qcow2 disk image in the above zip.

The username and password are both the word <strong>root</strong>.

For Linux users (and Mac users wanting to use the homebrew version), you can find on CourseWeb a test version of the disk image and a start.sh script to run it (qemu-test.zip). It should be identical to the above version in terms of functionality, but actually boot with a recent version of QEMU. IF THE ORIGINAL WORKS FOR YOU, DON’T BOTHER WITH THIS ONE.

On Mac OS X, if you don’t have Homebrew, open a terminal and type:

/usr/bin/ruby -e “$(curl -fsSL

<u><a href="https://raw.githubusercontent.com/Homebrew/install/master/install">https://raw.githubusercontent.com/Homebrew/install/master/install</a></u><a href="https://raw.githubusercontent.com/Homebrew/install/master/install">)</a>” Go through the install steps. When done, install qemu by typing:

brew install qemu

That will install qemu. Now you can run start.sh from the terminal <strong>in the unzipped folder</strong> to launch qemu.

On Linux, using your appropriate package manager, install qemu-system-arm, likely part of your distro’s qemu package.

Then run start.sh <strong>in the unzipped folder</strong> to launch qemu.

<h2><a name="_Toc7487"></a>Copying the Files to QEMU</h2>




From QEMU, you will need to download two files from the new kernel that you just built. The kernel itself is a file named bzImage that lives in the directory linux-2.6.23.1/arch/i386/boot/. There is also a supporting file called System.map in the linux-2.6.23.1/ directory that tells the system how to find the system calls.

Use scp to download the kernel to a home directory (/root/ if root):

scp <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="b1e4e2f4e3fff0fcf4f1c5d9dec5d99fd2c29fc1d8c5c59fd4d5c4">[email protected]</a>:/u/OSLab/USERNAME/linux-2.6.23.1/arch/i386/boot/bzImage .

scp <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="2f7a7c6a7d616e626a6f5b47405b47014c5c015f465b5b014a4b5a">[email protected]</a>:/u/OSLab/USERNAME/linux-2.6.23.1/System.map .

<h2><a name="_Toc7488"></a>Installing the Rebuilt Kernel in QEMU</h2>




As root (either by logging in or via su):

cp bzImage /boot/bzImage-devel cp System.map /boot/System.map-devel

and respond ‘y’ to the prompts to overwrite. Please note that we are replacing the -devel files, the others are the original unmodified kernel so that if your kernel fails to boot for some reason, you will always have a clean version to boot QEMU.

You need to update the bootloader when the kernel changes. To do this (do it every time you install a new kernel if you like) as root type:

lilo

lilo stands for LInux Loader, and is responsible for the menu that allows you to choose which version of the kernel to boot into.

<h2><a name="_Toc7489"></a>Booting into the Modified Kernel</h2>




As root, you simply can use the reboot command to cause the system to restart. When LILO starts (the red menu) make sure to use the arrow keys to select the linux(devel) option and hit enter.




<h2><a name="_Toc7490"></a>Building and Running test programs</h2>




If we try to build your test program using gcc, the &lt;linux/unistd.h&gt; file that will be preprocessed in will be the one of the kernel version that thoth.cs.pitt.edu is running and we will get an undefined symbol error. This is because the default unistd.h is not the one that we changed. What instead needs to be done is that we need to tell gcc to look for the new include files with the -I option:

gcc -m32 -o trafficsim -I /u/OSLab/USERNAME/linux-2.6.23.1/include/ trafficsim.c

We cannot run our test program on thoth.cs.pitt.edu because its kernel does not have the new syscalls in it. However, we can test the program under QEMU once we have installed the modified kernel. Start QEMU with the modified kernel and download the test program (e.g. trafficsim) using scp as we did for the kernel. We can just run the test program from our home directory without any installation necessary.

./trafficsim




<h2><a name="_Toc7491"></a>File Backups</h2>




One of the major contributions the university provides for the AFS filesystem is nightly backups. However, the /u/OSLab/ partition on thoth is not part of AFS space. Thus, any files you modify under your personal directory in /u/OSLab/ are not backed up. If there is a catastrophic disk failure, all of your work will be irrecoverably lost. As such, it is my recommendation that you:

<strong>Backup all the files you change under </strong><strong>/u/OSLab</strong> <strong>or</strong> <strong>QEMU</strong><strong> to your </strong><strong>~/private/</strong><strong> directory frequently!</strong>

BE FOREWARNED: Loss of work not backed up is not grounds for an extension.

<h2><a name="_Toc7492"></a>Notes</h2>

<ul>

 <li>printk() is the version of printf() you can use for debugging messages from the kernel.</li>

 <li>In general, you can use some library standard C functions, but not all. If they do an OS call, they may not work</li>

</ul>

<a href="#_ftnref1" name="_ftn1">[1]</a> Based upon Project 2 of Dr. Misurda’s CS 1550 course.