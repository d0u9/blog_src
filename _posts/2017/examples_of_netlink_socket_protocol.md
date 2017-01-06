---
title: 'Examples of netlink socket protocol'
categories:
  - Science & Technology
  - Linux
tags:
  - Linux
  - Linux Kernel
  - Netlink Socket
abbrlink: 2213062665
date: 2017-01-07 07:26:00
---

Excerpted from Wikipedia:

> Netlink socket family is a Linux kernel interface used for inter-process communication (IPC) between both the kernel and userspace processes, and between different userspace processes, in a way similar to the Unix domain sockets. Similarly to the Unix domain sockets, and unlike INET sockets, Netlink communication cannot traverse host boundaries. However, while the Unix domain sockets use the file system namespace, Netlink processes are addressed by process identifiers (PIDs).

From the description above, it is not hard to know that Netlink acts as a bridge between the kernel and userspace program. Netlink protocol is datagram-oriented and has its own communication protocol.

In this post, we will focuse on the NETLINK_ROUTE protocol -- a protocol of **AF\_NETLINK** family. All the supported netlink protocols can be found in **man(7) netlink**.

<!-- more -->

# Some useful references

1. [Understanding And Programming With Netlink Sockets](https://people.redhat.com/nhorman/papers/netlink.pdf)
2. [man(7) netlink](https://linux.die.net/man/7/netlink)
3. [man(3) netlink](https://linux.die.net/man/3/netlink)
4. [man(7) rtnetlink](https://linux.die.net/man/7/rtnetlink)
5. [man(3) rtnetlink](https://linux.die.net/man/3/rtnetlink)
6. [libnl](https://www.infradead.org/~tgr/libnl/doc/core.html)
7. [Linux Kernel source code - netlink](http://lxr.free-electrons.com/source/include/uapi/linux/netlink.h#L52)

# netlink

## netlink header structure

```C
struct nlmsghdr {
    __u32 nlmsg_len;    /* Length of message including header */
    __u16 nlmsg_type;   /* Type of message content */
    __u16 nlmsg_flags;  /* Additional flags */
    __u32 nlmsg_seq;    /* Sequence number */
    __u32 nlmsg_pid;    /* Sender port ID */
};
```

Netlink message contains one or more **nlmsghdr** structures, each of them has a associated payload. The payload is varied according to the netlink meesage type which is indicated by `nlmsg_type` field. In this post, we are mainly
concentrating on `NETLINK_ROUTE` message type.

## netlink Macros

As the **man(7) netlink** suggested, the netlink byte stream should be accessed only with the standard `NLMSG_*` macros, **man(3) netlink** for more details.

## A visual view of netlink message

To better illustrate the layout of netlink protocol, visual view is the best.

```
+----------+---------+------------------------+---------+ +----------+---------+
|          |         |                        |         | |          |         |
| nlmsghdr | padding |         payload        | padding | | nlmsghdr |   ...   |
|          |         |                        |         | |          |         |
+----------+---------+------------------------+---------+ +----------+---------+
```

Due to the fact that `staruct nlmsghdr` has already well aligned on some architecture (e.g. my x86_64 Linux), so the paddings in the picture above maybe not present in the real world.

# rtnetlink

**rtnetlink**, as described in **man(3) rtnetlink**, allows user to read and modify kernel's routing table. Each rtnetlink message contains one rtnetlink header, and followed by one or more attributes which is represented by `struct rtattr`.

## rtnetlink header structure

The header structure of **rtnetlink** is varied according to different message type.

We will mainly introduce `RTM_NEWADDR`, `RTM_DELADDR` and `RTM_GETADDR` types in this post to illustrate how to set and get IP address via Netlink protocol.

The `RTM_NEWADDR`,`RTM_DELADDR` and `RTM_GETADDR` message types have the same message header structure, i.e. `struct ifaddrmsg`.

```C
struct ifaddrmsg {
  unsigned char ifa_family;    /* Address type */
  unsigned char ifa_prefixlen; /* Prefixlength of address */
  unsigned char ifa_flags;     /* Address flags */
  unsigned char ifa_scope;     /* Address scope */
  int           ifa_index;     /* Interface index */
};
```

## attributes of `RTM_NEWADDR`, `RTM_DELADDR` and `RTM_GETADDR`

As dictated before, each rtnetlink message is made up of two parts -- one header and one or more optional attributes. The attributes contain the actual values which is passed via netlink. For example, when setting IP address, netlink header indicates that we opt rtnetlink message type to read and modify kernel's routing table (include IP address assigning), rtnetlink header designates that we are interested in IP manipulation, and the attributes carry the actual IP address value.

Attributes has the structure below:

```C
struct rtattr {
   unsigned short rta_len;    /* Length of option */
   unsigned short rta_type;   /* Type of option */
   /* Data follows */
};
```

For `RTM_NEWADDR`, `RTM_DELADDR` and `RTM_GETADDR` message type, there 7 available attribute types.

```
rta_type        value type             description
─────────────────────────────────────────────────────────────
IFA_UNSPEC      -                      unspecified.
IFA_ADDRESS     raw protocol address   interface address
IFA_LOCAL       raw protocol address   local address
IFA_LABEL       asciiz string          name of the interface
IFA_BROADCAST   raw protocol address   broadcast address.
IFA_ANYCAST     raw protocol address   anycast address
IFA_CACHEINFO   struct ifa_cacheinfo   Address information.
```

## A visual view of rtnetlink message

```
+---------------------------------------- NLMSG_LENGTH --------------------------------------+
|                                                                                            |
|                                          +-------- RTA_LENGTH ---------+                   |
|                                          |                             |                   |
+----------+---------+-----------+---------+--------+----------+---------+--------+----------+---------+----------+
|          |         |           |         |        |          |         |        |          |         |          |
| nlmsghdr | padding | ifaddrmsg | padding | rtattr | rta_data | padding | rtattr | rta_data | padding | nlmsghdr |
|          |         |           |         |        |          |         |        |          |         |          |
+----------+---------+-----------+---------+--------+----------+---------+--------+----------+---------+----------+
                     ^                     ^        ^                    ^                             ^
                     |                     |        |                    |                             |
                     |                     |        |                    |                             |
                 NLMSG_DATA             IFA_RTA   RTA_DATA            RTA_NEXT                     NLMSG_NEXT
```

# Example to illustate netlink usage

Here we post a simple example to illustrate the usage of netlink. The full code can be found on my [Github](https://www.github.com/d0u9/examples/blob/master/C/netlink/ip_add.c).

Download this code and compile it with `gcc ip_add.c`.

---

### ¶ The end
