

###[**System and kernel security**](http://source.android.com/security/overview/kernel-security.html)

At the operating system level, the Android platform provides the security of the Linux kernel, as well as a secure inter-process communication (IPC) facility to enable secure communication between applications running in different processes. These security features at the OS level ensure that even native code is constrained by the Application Sandbox. Whether that code is the result of included application behavior or a exploitation of an application vulnerability, the system would prevent the rogue application from harming other applications, the Android system, or the device itself.

Linux Security

The foundation of the Android platform is the Linux kernel. The Linux kernel itself has been in widespread use for years, and is used in millions of security-sensitive environments. Through its history of constantly being researched, attacked, and fixed by thousands of developers, Linux has become a stable and secure kernel trusted by many corporations and security professionals.

As the base for a mobile computing environment, the Linux kernel provides Android with several key security features, including:

A user-based permissions model
Process isolation
Extensible mechanism for secure IPC
The ability to remove unnecessary and potentially insecure parts of the kernel
As a multiuser operating system, a fundamental security objective of the Linux kernel is to isolate user resources from one another. The Linux security philosophy is to protect user resources from one another. Thus, Linux:

Prevents user A from reading user B's files
Ensures that user A does not exhaust user B's memory
Ensures that user A does not exhaust user B's CPU resources
Ensures that user A does not exhaust user B's devices (e.g. telephony, GPS, bluetooth)