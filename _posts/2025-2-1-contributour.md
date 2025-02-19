---
title: "ContribuTour"
excerpt: "Explore essential tools and workflows for contributing to open source projects."
seo_description: "Learn the tools, workflows, and best practices for effective open source contributions."
description: "A tour covering tools, workflows, and best practices for contributing to open source projects."
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
  bc bpfcc-tools bsdextrautils tldr-py trace-cmd tree tuna virt-what yad \
  clang coccinelle coreutils cppcheck cscope curl exuberant-ctags \
  fakeroot flawfinder git gnuplot hwloc indent kmod libnuma-dev \
  man-db net-tools numactl perf-tools-unstable procps psmisc \
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

To build a kernel quickly, I use the following commands:

```bash
# Clean everything
make mrproper
# Build only with currently loaded modules
# Remove all custom modules if any
make localmodconfig
# Add or remove specific options
make menuconfig
```

For cross-compiling on an SoC, use the same commands with a cross-compiler
prefix:

```bash
# For 32/64-bit
KERNEL=kernel8
# Copy the original configuration
scp <user>@<address>:/boot/config* .config
# Configure the kernel based on the original one
make ARCH=arm64 olddefconfig
make ARCH=arm64 menuconfig
```

[The Linux kernel](https://www.raspberrypi.com/documentation/computers/linux_kernel.html)
documentation for Raspberry Pi provides a thorough guide on kernel builds.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>There is a tool inside the Linux source code that allows you to change any
configuration with a single command:
`scripts/config --enable CONFIG_DEBUG_KERNEL`
{: .prompt-tip }
<!-- markdownlint-restore -->

I use the following script to enable many debugging capabilities inside the
kernel during the configuration:

```bash
#!/bin/bash
# Generic kernel debugging instruments
scripts/config --enable CONFIG_DEBUG_KERNEL
scripts/config --enable CONFIG_DEBUG_INFO
scripts/config --enable CONFIG_DEBUG_MISC
scripts/config --enable CONFIG_MAGIC_SYSRQ
scripts/config --enable CONFIG_DEBUG_FS
scripts/config --enable CONFIG_KGDB
scripts/config --enable CONFIG_UBSAN
scripts/config --enable CONFIG_KCSAN
# Memory debugging
scripts/config --enable CONFIG_DEBUG_PAGEALLOC
scripts/config --enable CONFIG_DEBUG_PAGEALLOC_ENABLE_DEFAULT
scripts/config --enable CONFIG_SLUB_DEBUG
scripts/config --enable CONFIG_DEBUG_MEMORY_INIT
scripts/config --enable CONFIG_KASAN
scripts/config --enable CONFIG_DEBUG_SHIRQ
scripts/config --enable CONFIG_SCHED_STACK_END_CHECK
scripts/config --enable CONFIG_DEBUG_PREEMPT
# Lock debugging
scripts/config --enable CONFIG_PROVE_LOCKING
scripts/config --enable CONFIG_RCU_EXPERT
scripts/config --enable CONFIG_LOCK_STAT
scripts/config --enable CONFIG_DEBUG_RT_MUTEXES
scripts/config --enable CONFIG_DEBUG_MUTEXES
scripts/config --enable CONFIG_DEBUG_SPINLOCK
scripts/config --enable CONFIG_DEBUG_RWSEMS
scripts/config --enable CONFIG_DEBUG_LOCK_ALLOC
scripts/config --enable CONFIG_DEBUG_ATOMIC_SLEEP
scripts/config --enable CONFIG_PROVE_RCU_LIST
scripts/config --enable CONFIG_DEBUG_OBJECTS_RCU_HEAD
scripts/config --enable CONFIG_BUG_ON_DATA_CORRUPTION
scripts/config --enable CONFIG_STACKTRACE
scripts/config --enable CONFIG_DEBUG_BUGVERBOSE
scripts/config --enable CONFIG_FTRACE
scripts/config --enable CONFIG_FUNCTION_TRACER
scripts/config --enable CONFIG_FUNCTION_GRAPH_TRACER
# Arch specific
scripts/config --enable CONFIG_FRAME_POINTER
scripts/config --enable CONFIG_STACK_VALIDATION
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>While inside `make menuconfig`, you can use `/` to search for a configuration.
Then, press `1` (or the appropriate number) to go from the search screen to the
menu item for the first entry found.
{: .prompt-tip }
<!-- markdownlint-restore -->

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>Make sure the `CONFIG_SYSTEM_REVOCATION_KEYS` is empty.
{: .prompt-tip }
<!-- markdownlint-restore -->

### Build the Kernel

This is the easy part:

```bash
# Actual build
make -j$(nproc) all
```

When cross-compiling:

```bash
# Actual build
make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- all
# Package the generated artifacts
make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- deb-pkg
# Copy the new kernel to the SoC for installation later
cd ..
scp *.deb <user>@<address>:/home/<user>
```

### Install the Kernel

For a native build:

```bash
# Install modules
sudo make modules_install
# Install the kernel and update GRUB
sudo make install
```

Or on your SoC:

```bash
# Install the new kernel
sudo dpkg -i *.deb
```

If GRUB is your bootloader, you can modify its behavior by editing the file at `/etc/default/grub`:

```bash
# Example edits:
GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=3
GRUB_SAVEDEFAULT=true
GRUB_DEFAULT=saved
GRUB_CMDLINE_LINUX=""
```

Don't forget to run `sudo update-grub`. Below are some useful kernel command-line
arguments:

- To enable kernel debug messages: `debug`
- To output loglevels less than `n`: `loglevel=n`
- To let the kernel be preempted: `preempt=full`
- To isolate CPUs from the scheduler: `isolcpus=0,4-8`
- To boot into single-user mode: `single`

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>If U-Boot is your bootloader, connect your SoC's serial port to your PC. During
boot, press any key on the keyboard to access the U-Boot command line. Once
there, you can add kernel command-line arguments like this:
`setenv extraargs ${extraargs} systemd.unit=rescue.target; env print;`
{: .prompt-tip }
<!-- markdownlint-restore -->

## Why Patch Files

Patch files are portable and plain text, which means the community doesn't need
to rely on a provider like GitHub to track branches and merge pull requests. When
patch files are sent as plain text emails, the review and back-and-forth
communication can also occur with any email client.

When your testing is complete and you have committed your work, create a patch
file with:

```bash
git format-patch origin/master -o ../
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>See the guidelines for [The canonical patch format](https://docs.kernel.org/process/submitting-patches.html#the-canonical-patch-format)
and how to [Describe your changes](https://docs.kernel.org/process/submitting-patches.html#describe-your-changes).
{: .prompt-tip }
<!-- markdownlint-restore -->

## Checking for Style

Make sure to [Style-check your changes](https://docs.kernel.org/process/submitting-patches.html#style-check-your-changes)
before submitting any patches. To style-check your changes without creating any
patch files:

```bash
git checkout feat
git format-patch origin/master --stdout | ./scripts/checkpatch.pl
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>Use `git commit --amend -s` to sign your commit.
{: .prompt-tip }
<!-- markdownlint-restore -->

## Finding Maintainers

We use another tool in the `scripts` subfolder of the Linux source code to find
relevant maintainers for our patch. This tool checks which files have been
modified in the commit and prints the maintainers' names and email addresses.

```bash
git checkout feat
git format-patch origin/master --stdout | ./scripts/get_maintainer.pl
```

## Submit Your Patch

All that to just send an email! Don't worry, Git has you covered:

```bash
git send-email --to=maintainer1@linux.com --cc=maintainer2@linux.com \
  --cc=maintainer3@linux.com master..feat
```

Before sending emails with Git, you need to install `git-email` and configure it
with your email provider's server information. The following shows how to do it
for Gmail:  

```bash
# Install git-email
sudo apt-get install git-email
# Use Gmail as the SMTP server
git config --global sendemail.smtpEncryption tls
git config --global sendemail.smtpServer smtp.gmail.com
git config --global sendemail.smtpUser yourname@gmail.com
git config --global sendemail.smtpServerPort 587
```

It's not over yet. You need to enable two-step verification on Gmail and generate
an [Application Password](https://security.google.com/settings/security/apppasswords)
to use when Git asks for it.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>Have a look at the [git-send-email](https://git-scm.com/docs/git-send-email)
documentation.
{: .prompt-tip }
<!-- markdownlint-restore -->

## Sending a v2 of Your Patch

It is very likely that your patch will not be accepted on the first submission.
When you receive an email with comments from the maintainer, apply those changes
to your commits and resend the patch files. Do not create new commits. Instead,
use [Interactive Rebase]({% link _posts/2024-05-19-eat-git-part-3.md %}#interactive-rebase).
Once done, use the `--annotate` parameter when sending the emails and manually
edit the subject line to add `v2` (and so on) to the `[Patch]` part.

```bash
git send-email --annotate --to=maintainer1@linux.com --cc=maintainer2@linux.com \
  --cc=maintainer3@linux.com master..feat
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>See [The canonical patch format](https://docs.kernel.org/process/submitting-patches.html#the-canonical-patch-format)
for more instructions.
{: .prompt-tip }
<!-- markdownlint-restore -->

## Writing Cover Letters

Sometimes you need to provide explanations that don't belong in the commit
message but can make your decisions in the code clearer to others. You can
instruct `git-send-email` to send an extra email before any patches, known as a
cover letter. To do that, add the `--cover-letter` parameter when sending the
emails. Replace `*** SUBJECT HERE ***` and `*** BLURB HERE ***` with the
appropriate content.

```bash
git send-email --annotate --cover-letter --to=maintainer1@linux.com \
  --cc=maintainer2@linux.com --cc=maintainer3@linux.com master..feat
```

## Replying to Emails

The maintainer will open your patch in their email client, add comments between
the lines of your patch, and reply in plain text format.

```
> > > +
> > > +/**
> > > + * usb4_disable_lrf() - Disable Link Recovery flow up to host router
> > > + * @sw: Router to start
> > > + *
> > > + * Disables Link Recovery flow from @sw up to the host router.
> > > + * Returns true if any Link Recovery flow was disabled. This
> > > + * can be used to figure out whether the link was setup by us
> > > + * or the boot firmware so we don't accidentally enable them if
> > > + * they were not enabled during discovery.
> >
> > Okay I think you copied the CLx part here, no? How did you test this?
> >
>
> The way we should handle Link Recovery seems quite similar to CL states.
```

That's why regular email clients are not ideal for viewing these emails, and you
should avoid replying to them using services like Gmail's web application.
Instead, install a terminal-based email client. One example is `neomutt`, which I
will explain how to set up.

<p align="center"><img src="https://i.postimg.cc/Gp17Z3DY/temp-Image-Phhqki.avif" alt="Dependency Graph"/></p>

To properly set up `neomutt`, create two files: `~/.config/mutt/muttrc` and
`~/.config/mutt/colors.muttrc`, with the following content:

```bash
# .muttrc
set imap_user = 'yourname@gmail.com'
set imap_pass = 'your_app_password'
set spoolfile = imaps://imap.gmail.com/INBOX
set folder = imaps://imap.gmail.com/
set record="imaps://imap.gmail.com/[Gmail]/Sent Mail"
set postponed="imaps://imap.gmail.com/[Gmail]/Drafts"
set trash="imaps://imap.gmail.com/[Gmail]/Trash"
set mbox="imaps://imap.gmail.com/[Gmail]/All Mail"
mailboxes =INBOX =[Gmail]/Sent\ Mail =[Gmail]/Drafts =[Gmail]/Spam =[Gmail]/Trash =[Gmail]/All\ Mail
set smtp_url = "smtp://yourname@smtp.gmail.com:587/"
set smtp_pass = $imap_pass
set ssl_force_tls = yes
set edit_headers = yes
set charset = UTF-8
unset use_domain
set realname = "Your Name"
set from = "yourname@gmail.com"
set use_from = yes
set envelope_from=yes
source colors.muttrc
```

For `colors.muttrc`, copy the content from [here](https://gist.githubusercontent.com/LukeSmithxyz/de94948264649a9264193e96f5610c44/raw/d274199d3ed1bcded2039afe33a771643451a9d5/colors.muttrc). Then run the `neomutt`.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> It takes some time to get used to `neomutt` shortcuts. The following is a list
of the most common ones I use:
- `c` to change mailbox
- `space` to go down one page
- `-` To go up one page
- `Enter/Return` to go down one line
- `Backspace/Delete` to go up one line
{: .prompt-tip }
<!-- markdownlint-restore -->

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> To see a patch email with proper text highlighting:  
1. Choose an email containing a patch. Use the 'Enter/Return' key to open it.
2. Press `v` to view attachments.
3. Press `|` to pipe the content to `vim`.
4. Use `vim -` when asked with the `Pipe to:` question.
5. In `vim`, use the `:set filetype=diff` command to highlight the text properly.
6. Use `:qa!` to quit.
{: .prompt-tip }
<!-- markdownlint-restore -->

## Ideas on Where to Start

Follow the [discussions](https://subspace.kernel.org/vger.kernel.org.html)
between developers to find the hotspots of development in the Linux kernel. If
you are familiar with a protocol, find the latest specification and
cross-reference the source code with the requirements of that specification.
Drivers are updated continuously, so they are a good place to start. There are
many GNU projects out there—follow their bug reporting channels and see if you
can fix any of them. It takes some time to find your footing, but once you do,
you'll find this whole process to be a very rich learning experience.
