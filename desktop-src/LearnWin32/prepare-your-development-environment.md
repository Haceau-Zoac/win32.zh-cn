---
title: 准备好你的开发环境
description: 准备好你的开发环境
ms.topic: article
ms.date: 14/05/2021

---

# 准备好你的开发环境

在使用 C 或是 C++ 编写 Windows 程序之前，你必须安装 Microsoft Windows 软件开发工具包（SDK）或 Microsoft Visual Studio。Windows SDK 包括编译和链接你的程序所必要的头文件和库。Windows SDK 还有一些用于编译 Windows 程序的命令行工具，包括 Visual C++ 编译器和链接器。尽管你可以使用命令行工具来编译和构建 Windows 程序，但我们还是推荐使用 Microsoft Visual Studio 完成这一切。你可以下载免费的 Visual Studio 社区版或是在[这里](https://visualstudio.microsoft.com/downloads/)下载免费试用的 Visual Studio 其他版本。

Windows SDK 的每个版本都针对最新版本的 Windows 以及以前的几个版本。发行说明列出了所有支持的平台，但除非你的程序要一个非常老的 Windows 版本，你应该安装最新的 Windows SDK。你可以在[此](https://developer.microsoft.com/windows/downloads/windows-10-sdk)安装最新的 Windows SDK。

Windows SDK 支持 32 位和 64 位程序。Windows API 的设计让同一份代码可以同时在 32 位和 64 位下编译并无需任何改动。

> [!Note]  
> Windows SDK 不支持硬件驱动开发，并且此系列不会讨论驱动开发。有关编写硬件驱动的更多信息，请参见 [Windows 驱动入门](/windows-hardware/drivers/gettingstarted/)。

## 下一步

[Windows 编码习惯](windows-coding-conventions.md)

## 相关主题

* [下载 Visual Studio](https://visualstudio.microsoft.com/downloads/)

* [下载 Windows SDK](https://developer.microsoft.com/windows/downloads/windows-10-sdk)