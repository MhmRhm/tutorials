---
title: "ContribuTour"
excerpt: "Explore essential tools and workflows for contributing to open source projects."
seo_description: "Learn the tools, workflows, and best practices for effective open source contributions."
description: "A practical guide covering tools, workflows, and best practices for contributing to open source projects."
categories: [Open Source, Linux]
tags: [open-source, linux, git, github, contribution]
pin: true
date: 2025-02-01
image:
  path: /assets/img/Tux.svg
  alt: "Guide to tools and workflows for open source contributions"
---

## Preface

When I first started with open-source, I was confused. What are patch files? Who
should I send them to? How do I send them? Where do developers communicate? And
why do those emails look so cryptic?

These tools and workflows form a culture among open-source contributors. In this
short tour, I will clear up these confusions. If you're in a similar situation,
you may find it useful. My focus is on the Linux kernel source code—the largest
and most professional open-source ecosystem. Many other projects have adopted
parts of its workflow.

There are many tutorials online, with the original one being [All development-process docs](https://docs.kernel.org/process/index.html#).
However, none present the essentials concisely with examples. So, let's begin
this tour.

### What You Won't Learn Here

1. Programming
2. Git
3. Debugging

We won't cover these topics, as there are countless resources available. For
example, [Linux Kernel Programming](https://www.packtpub.com/en-us/product/linux-kernel-programming-9781803232225)
and [Linux Kernel Debugging](https://www.packtpub.com/en-us/product/linux-kernel-debugging-9781801076753)
are excellent references.

I also have two tutorials on Git and GDB called [EatGit]({% link _posts/2024-05-08-eat-git-part-1.md %})
and [GDBugging]({% link _posts/2024-12-8-gdbugging-101.md %}). Feel free to check
them out if you're interested.

### What You Will Learn Here

0. Proper Setup
1. Why Patch Files
2. Checking for Style
3. Finding Maintainers
4. Sending Emails
5. Replying to Emails
6. Sending a v2 of Your Patch
7. Writing Cover Letters

Here, I will take you through what happens after you've finished coding—when you
want to share your work with the community and receive feedback.

## Proper Setup

After editing your code, you need to build and test it. Each codebase has its own
methods, usually documented. For the Linux kernel, `make` and its various build
targets handle OS configuration, building, and installation. Below is a concise
guide to building the Linux kernel.

1. Set Up a Test Machine
2. Install Build Dependencies
3. Configure the Kernel
4. Build the Kernel
5. Install the Kernel

### Set Up a Test Machine

The setup depends on the part of the code you're testing. If you're working on a
device driver, you'll need a machine that can access the target device, making
virtual machines less suitable. You may want to explore the
[Armbian Linux Build Framework](https://github.com/armbian/build),
[The Yocto Project](https://www.yoctoproject.org), or
[Buildroot](https://buildroot.org).

For native builds, I set up a clean [Ubuntu Server](https://ubuntu.com/download/server)
virtual machine and connect to it using the [VSCode Remote Development](https://code.visualstudio.com/docs/remote/remote-overview)
extension. After installing dependencies and cloning the Linux kernel repository,
I save the setup and power off the machine. From then on, I clone this machine
whenever needed and work inside the clone.

When building for my OrangePi 5, I cross-compile the kernel, move it to the
OrangePi boot directory, and update the symbolic links.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>[The kernel’s command-line parameters](https://docs.kernel.org/admin-guide/kernel-parameters.html)
list all the arguments the kernel accepts. To see the arguments of the current
run, use `cat /proc/cmdline`.
{: .prompt-tip }
<!-- markdownlint-restore -->

### Install Build Dependencies

[Linux Kernel Programming](https://www.packtpub.com/en-us/product/linux-kernel-programming-9781803232225)
provides a useful script to set up all the dependencies needed for the topics
covered in the book. Here is a portion of it:

```bash
sudo apt-get update
sudo apt-get upgrade

# ensure basic pkgs are installed!
sudo apt-get install -y gcc make perl

# packages typically required for kernel build
sudo apt-get install -y \
  asciidoc binutils-dev bison build-essential flex libncurses5-dev ncurses-dev \
  libelf-dev libssl-dev openssl pahole tar util-linux xz-utils zstd

# other packages...
sudo apt-get install -y \
  bc bpfcc-tools bsdextrautils tldr-py trace-cmd tree tuna virt-what yad
  clang coccinelle coreutils cppcheck cscope curl exuberant-ctags \
  fakeroot flawfinder git gnuplot hwloc indent kmod libnuma-dev \
  man-db net-tools numactl perf-tools-unstable procps psmisc  \
  linux-headers-$(uname -r) linux-tools-$(uname -r) \
  rt-tests smem sparse stress stress-ng sysfsutils

# clean up
sudo apt-get autoremove
```

And finally, clone a suitable codebase from [Kernel.org git repositories](https://git.kernel.org):  

```bash
git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
```

### Configure the Kernel



### Build the Kernel
### Install the Kernel

## Why Patch Files
## Checking for Style
## Finding Maintainers
## Sending Emails
## Replying to Emails
## Sending a v2 of Your Patch
## Writing Cover Letters
