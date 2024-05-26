---
title: Contribution to Kworkflow
date: 2024-05-06
categories: [University of São Paulo (USP), MAC5856 - Desenvolvimento de Software Livre]
tags: [software livre, open source]     # TAG names should always be lowercase
comments: false
---

Comments on my contribution to kworkflow, a kernel developer workflow tool.

### Motivation & Description

In the context of the course I'm taking this semester, we were encouraged to contribute to the [Linux kernel](https://gustavohenriquesr.github.io/posts/MAC5856_Fase_2_kernel/) and the tools associated with its development. In this way, we saw the possibility of contributing to [kworkflow](https://kworkflow.org/), a kernel development tool intended to provide support for daily tasks. So, Bruna and I chose to work on an [issue](https://github.com/kworkflow/kworkflow/issues/951#issue-1977355255) that has been open for a while, which is about improving the amount of information on the development machine.

In more specific terms, we want to improve the output of the [`kw device`](https://kworkflow.org/man/features/kw-device.html) command, which provides general information about the machine and the software under development, such as the amount of ram, chassis, cpu, operating system, motherboard, among others. We therefore propose, as was the intention of the issue, to add information about the kernel (such as name, release, version and machine hardware name) and about the displays on the tool user's machine (number of active displays, display name, resolution, etc.). 

### What has been done (so far)

Since the task imposed in the issue can be divided into two parts, Bruna was responsible for solving the part about displays, while I would do the part about information related to the kernel. With that, we opened a [Pull Request](https://github.com/kworkflow/kworkflow/pull/1107) with the suggested changes, which was promptly reviewed by [Rodrigo Siqueira](https://github.com/rodrigosiqueira).

With regard to the kernel part, the first revision suggested changing the documentation of the added function (`get_kernel_info()`) in the file *device_info.sh* to make it clearer to read, adding more information about kernel (about kernel version and machine hardware name, since initially I only added kernel name and release), and correcting the project's style compatibility (changing the use of quotation marks and the use of terminal commands with long options). Finally, for this case, it was also requested that unit and integration tests be added to verify the changes made.

In this sense, I made the suggested changes that I believed were appropriate and made changes to the unit test, trying to preserve the coherence of the style of the files and trying to keep it as clear as possible. Below you can see a part of the function to obtain information about the kernel, after the first revision. Now, I'm waiting for a second review from Siqueira in order to continue the work.

```c
2) # LOCAL_TARGET
    show_verbose "$flag" "$cmd_name"
    kernel_name=$(cmd_manager 'SILENT' "$cmd_name")

    show_verbose "$flag" "$cmd_release"
    kernel_release=$(cmd_manager 'SILENT' "$cmd_release")

    show_verbose "$flag" "$cmd_version"
    kernel_version=$(cmd_manager 'SILENT' "$cmd_version")

    show_verbose "$flag" "$cmd_machine"
    kernel_machine=$(cmd_manager 'SILENT' "$cmd_machine")
    ;;
```