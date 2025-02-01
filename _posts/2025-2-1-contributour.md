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

There are many tutorials online, with the original one being
[All development-process docs](https://docs.kernel.org/process/index.html#).
However, none present the essentials concisely with examples. So, let's begin
this tour.

### What You Won't Learn Here

1. Programming
2. Git
3. Debugging

We won't cover these topics here, as countless resources are available for
learning them. In fact, I have two tutorials on Git and GDB called
[EatGit]({% link _posts/2024-05-08-eat-git-part-1.md %}) and
[GDBugging]({% link _posts/2024-12-8-gdbugging-101.md %}). Feel free to check
them out if you're interested.

### What You Will Learn Here

0. Proper Setup
1. Why Patch Files Exist
2. Checking for Style
3. Finding Maintainers
4. Sending Emails
5. Replying to Emails
6. Sending a v2 of Your Patch
7. Writing Cover Letters

Here, I will take you through what happens after you've finished coding—when you
want to share your work with the community and receive feedback.

