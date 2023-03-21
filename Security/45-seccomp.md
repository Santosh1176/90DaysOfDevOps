# Understanding Seccomp in Linux: An Introduction to Secure Computing Mode


Linux is an open-source operating system widely used in servers, supercomputers, and embedded systems. As with any operating system, security is a crucial aspect of Linux to protect against unauthorized access, data theft, and other security breaches. Linux has a reputation for being a secure operating system due to its open-source nature, strong community support, and regular security updates. However, it is essential to put in place security best practices to protect against potential vulnerabilities and keep the system secure.

Among many security features Linux comes bundled with User and permission management, Firewalls, Auditing and logging, Secure boot, and Malware protection. Apart from these and many other features, there are some advanced security tools that help us enhance the security posture and implement granular security primitives.

In this post, I'll be discussing one among many security tools Linux ships with, that helps us in securing applications running on a Linux host —  **Seccomp**.

While Linux already has a strong permission-based system that restricts access to resources, Seccomp is an important security feature of Linux that is used to provide additional layers of protection to the system. By using Seccomp, a system can better protect itself from zero-day exploits and other advanced attacks that may exploit vulnerabilities in the system.

Let's try to understand what Seccomp is in the following lines. However, to understand these Linux security features. Let's understand system calls in Linux on which Seccomp and many other Security tools are built upon.

# System Calls — syscalls



Whenever we run a command on a Linux machine, multiple system calls, or *syscalls* are triggered. Each syscall takes a command from a human user and passes it to the Linux kernel. These syscalls tell the Linux kernel, that it needs to perform some sort of privileged action. Opening or closing files, writing to files, or changing file permissions or ownership are just a few of the actions that require making some syscall. To see this in action, we can use the `strace` command to see the list of syscalls made by any command or process in Linux. For example:

```bash
santosh@~:$ strace -c ls
<snip>
.
.
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 26.99    0.000251          14        17           write
 17.74    0.000165           9        18           mmap
 11.72    0.000109          15         7           openat
  9.03    0.000084           9         9           close
  8.28    0.000077          38         2           getdents64
  4.62    0.000043           7         6           mprotect
.
.
  0.00    0.000000           0         1           execve
------ ----------- ----------- --------- --------- ----------------
100.00    0.000930           9        94         5 total
```
From the above snippet, we can see a simple command `ls` made close to one hundred syscalls to the kernel to fetch the contents of the home directory.

Irrespective of whether the application runs on a Linux container or a Linux host, the application makes several syscalls to the same kernel. There might be some applications that do not need to make any syscalls, and there might be some applications that might be making some system calls that are not required by the application code itself. Hence, following the principle of **least privileges** there are some Linux security features that allow users to limit the set of system calls that different programs can access.



## Seccomp — Secure Computing

Secure Computing or Seccomp in short, originally created in 2005 allows you to either enable just a certain subset of syscalls that you want for a process to use or disable certain syscalls that you want to prevent a process from using. The basic idea behind Seccomp is, once a process is started in Seccomp mode, that process would be able to make a very limited set of syscall — fewer syscalls results in a smaller attack surface. This feature of adding syscall filtering was added to the kernel known as `seccomp-bpf` in 2012.  This approach used *Berkley Packet Filters* to determine whether or not a given system call is permitted, based on a Seccomp profile applied to the process. 

By default, the Linux kernel allows any syscalls to be invoked by the program running inside the userspace. Hence, this feature of limiting the number of syscalls a process can invoke helps us in reducing the attack surface. However, implementing this feature comes with some challenges. Limiting some syscalls that the process needs to function properly needs to be identified and allowed else, this important security feature would become an impediment to the functioning of the application.

We can check if the kernel supports Seccomp by using `grep CONFIG_SECCOMP= /boot/config-$(uname -r)` command and we should expect `CONFIG_SECCOMP=y`, here  **y** indicates that Seccomp is enabled:

```bash
grep CONFIG_SECCOMP= /boot/config-$(uname -r)
CONFIG_SECCOMP=y
```
Seccomp when applied using profiles operates with three modes:

- **Mode 0**: Also known as **Unconfined**. This means Seccomp is disabled and all the syscalls are allowed by default.

- **Mode 1**: In this mode, Secomp is enabled with strict mode. Allowing only four syscalls i.e. `read`, `write` `exit`, and `sigreturn`, and all other syscalls are blocked.

- **Mode 3** With this mode, Seccomp is allowed with Selective Filtering of syscalls to be allowed.

To check the mode in which our system is running, we can apply the following command and see the result:

```bash
santosh@~:$ grep -i seccomp /proc/1/status
Seccomp:    0
Seccomp_filters:    0
```
By now, as we know *by default, the Linux kernel allows any syscalls to be invoked by the program running inside the userspace*. Using the above command, we can see that the Seccomp is disabled and no Seccomp filters are been deployed. on my system. This implies any system calls are allowed to be invoked by the userspace. 

Most container tools like Docker implement Seccomp in an effective way maintaining a balance between tightening the security while remaining portable to allow most workloads to run without receiving permission errors. The safest way to see Seccomp in action is by implementing it in an isolated environment, like Docker Containers. We can try running a ubuntu container and check its Seccomp mode:

```bash
santosh@~:$ docker run -it --rm ubuntu /bin/sh
# grep -i seccomp /proc/1/status
Seccomp:    2
Seccomp_filters:    1
```

You can see from the above snippet, Seccomp is enabled in `strict mode` on this container. We can further see a parameter called `Seccomp_filters: 1`, which means that the process has one seccomp filter installed. This filter restricts the system calls that the process can make to a pre-defined set of allowed system calls. 

Let's try to understand what Seccomp filters are,

## Seccomp Filters and Profiles

As we've discussed earlier, every program that runs on a computer uses something called system calls to communicate with the computer's operating system. However, most programs only use a small subset of the available system calls, and some of these system calls may have bugs that could be exploited by attackers.

To reduce the risk of these bugs being exploited, some programs benefit from only having access to a reduced set of system calls. This means that the program can only communicate with the operating system using a limited number of pre-approved commands, rather than having access to all the possible commands. This helps to make the program more secure by reducing the amount of communication between the program and the operating system.

By default, Docker uses a [built-in Seccomp profile](https://docs.docker.com/engine/security/seccomp/#significant-syscalls-blocked-by-the-default-profile) when we create a container. A seccomp profile is a simpler and more general way of implementing seccomp filtering. To see this in action, let's take a look at a simple profile. if we try to change the date from inside the ubuntu container, the request would be rejected as it invokes the `ckock_adjtime` syscall, which is disallowed by the seccomp profile used for this container.

```bash
# date -s '15 MAR 2023 22:00:00'
date: cannot set date: Operation not permitted
Wed Mar 15 22:00:00 UTC 2023
# 
```
As anticipated, the request to change the date was rightly rejected. 

Now, let us build a simple but very restrictive custom Seccomp profile as below"
```json
#Blacklist-seccomp-profile.json
{
    "defaultAction": "SCMP_ACT_ALLOW",
    "architectures": [
        "SCMP_ARCH_X86_64"
    ],
    "syscalls": [
        {
            "names":[
                "ckock_adjtime",
                "mkdir"
            ],
            "action": "SCMP_ACT_ERRNO"
        }
    ]
}
```         
The above snippet shows a sample profile, one of the important information is the `architectures` field. Some computer systems support different ways of making system calls, which can lead to different system call numbers being used for the same action. And, as system call filters rely on specific system call numbers to identify and control access to specific system calls, so if the system call numbers vary between different calling conventions, the filters may not work as intended.

The above profile also shows syscalls that are denied if this filter is enforced defined in `syscalls.name` array which has an `action` defined as `SCMP_ACT_ERRNO`.In our example, I have disallowed syscalls `clock_adjtime` and `mkdir` system calls. This would implicitly deny changing the system time and creating new directories on this container. Any other system calls invoked by the application will be allowed as defined in the `defaultAction` clause with the `SCMP_ACT_ALLOW` value.

```bash
santosh@blogs:main$ docker run -it --security-opt \          
     seccomp=./seccomp-test.json ubuntu bash
root@404d1a11ad9a:/# date -s '15 MAR 2023 22:00:00'
date: cannot set date: Operation not permitted
Wed Mar 15 22:00:00 UTC 2023

root@404d1a11ad9a:/# mkdir test
mkdir: cannot create directory 'test': Operation not permitted
root@404d1a11ad9a:/# mkdir -p ./tmp/test
mkdir: cannot create directory './tmp/test': Operation not permitted
```
The above profile in the example is a Blacklist profile. This blacklists some system calls that we identified might cause some security risks to our application. This kind of profile is much easier to define, but we have to iterate over to identify the system calls that can pose some security challenges and add them to the blacklist `SCMP_ACT_ERRNO` array. Another type of profile would be the Whitelist profile, as you might have guessed, it's the opposite of the Blacklist profile. Wherein, we define the system calls which are to be allowed and all other syscalls would be denied as defined in the below Blacklist profile:

```json
#Whitelist-seccomp-profile.json
{
    "defaultAction": "SCMP_ACT_ERRNO",
    "architectures": [
        "SCMP_ARCH_X86_64"
    ],
    "syscalls": [
        {
            "names":[
                "read",
                "write",
                "execve",
                .
                .
                .
                .
                "sigreturn",
                "exit"
            ],
            "action": "SCMP_ACT_ALLOW"
        }
    ]
}
```
This kind of profile might be too restrictive, as only some pre-defined syscalls are allowed. This needs the administrator to trace all the system calls made by the application and include them in the whitelist defined in the `syscalls` array. 

## Conclusion

We just scratched the surface of one of Linux's powerful security features — Seccomp, which helps improve the security of programs running on the system. It restricts the system calls that a program can make, thereby reducing the surface area for potential attacks.

When used in conjunction with Linux Containers, Seccomp can help provide an extra layer of isolation and protection for containerized applications. By applying Seccomp filters to a container, the attack surface of that container can be further reduced, and the potential impact of an attack can be minimized.

However, it's important to note that Seccomp is just one of many tools that can be used to improve the security of a system. It's not a silver bullet, and it should be used in conjunction with other Linux security tools, such as [Linux Security Modules (LSM)](https://en.wikipedia.org/wiki/Linux_Security_Modules), firewalls, etc.

