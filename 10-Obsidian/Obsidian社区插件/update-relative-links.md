---
uid: 20240227165413
title: Obsidian 插件：自动更新相对路径
tags: [Obsidian, 插件，官方增强]
description: 本插件主要弥补Obsidian官方设置“始终更新内部链接”的不足,可以自动更新相对路径
author: ImmortalSty
type: other
draft: false
editable: false
modified: 20240401125132
---

# Obsidian 插件：自动更新相对路径

## 概述

本插件主要弥补 Obsidian 官方设置“始终更新内部链接”的不足。

> [!Note] 插件名片
> - 插件名称：Update Relative Links
> - 插件作者：val
> - 插件说明：更新相对链接
> - 插件分类：官方增强
> - 项目地址：[点我访问](https://github.com/val3344/obsidian-update-relative-links)
> - 国内下载地址：[下载安装](https://pkmer.cn/products/plugin/pluginMarket/?update-relative-links)

## 插件作用

Obsidian 自带的设置 " 始终更新内部链接 "（Automatically update internal links） 选项实现的有问题。文件移动后，只会自动更新指向这个文件的链接，不会更新这个文件内指向其他文件的链接。这个插件就是用来解决这个问题的。另外还提供了一个批量修正所有已经存在的链接的命令。

例如有两个文件如下：

`dirA/fileA`:

```markdown
[fileB](../dirB/fileB.md)
```

`dirB/fileB`:

```markdown
[fileA](../dirA/fileA.md)
```

将 `fileA` 从 `dirA` 移动到 `dirB` 后：

`dirB/fileA`:

```markdown
[fileB](../dirB/fileB.md)
```

`dirB/fileB`:

```markdown
[fileA](fileA.md)
```

这个插件的作用，就是将 `dirB/fileA` 中的 `[fileB](../dirB/fileB.md)` 修复成 `[fileB](fileB.md)`。

## 使用方法

插件包含两个功能：

第一个功能：文件移动后，自动修复链接的相对路径。安装并启用插件后，移动文件会自动触发，无需其他操作。

第二个功能：批量将所有文件中的链接修复为相对路径。打开命令面板（默认快捷键是 `Ctrl + P`），搜索“Update all relative links”，回车执行。

## 插件限制

必须关闭 `Use [[Wikilinks]]` 选项，使用 Markdown 的链接语法。使用相对路径的意义也是为了让笔记更通用，如果还使用 Wiki Links，就没必要使用相对路径了。

## 为什么使用相对路径

Obsidian 的一大优势就是笔记完全由 Markdown 文件组成，不用担心迁移的问题。使用相对路径，可以使笔记有更好的通用性。例如使用其他 Markdown 编辑器查看、编辑笔记。如果使用 Git 同步笔记，还可以在网页上直接查看笔记。如果不使用相对路径，笔记中的链接，将只能在 Obsidian 中被识别。

## 进阶用法

配合 [[obsidian-linter_readme|Linter]] 插件 可以批量实现在相对链接的路径前加上 `./` 符号。

这么做可以把文件连接以相对路径进行，而不是以根目录路径为基准。

举个例子：

```
根目录
├── 文件夹 A
│   ├── 甲.md
│   └── 文件夹 B
│       └── 乙.md
└── 文件夹 B
    └── 乙.md
```

文件夹 A 中有一个名为甲的 Markdown 文件和一个文件夹 B，文件夹 B 中有一个名为乙的 Markdown 文件，同时根目录中还有一个同名的文件夹 B，其中也有一个名为乙的 Markdown 文件。此时，如果 A 中有链接 `[乙](文件夹 B/乙.md)`，这个链接会指向根目录中的乙文件（即 `/文件夹 B/乙.md`），而不是文件夹 A 中的乙文件（即 `/文件夹 A/文件夹 B/乙.md`）。

这是因为 Obsidian 会优先把 `文件夹 B/乙.md` 识别为基于库的绝对路径，而非相对路径。

如果想要让其指向文件夹 A 中的乙文件，则需要把链接改为 `[乙](./文件夹 B/乙.md)`，也就是在链接地址的前方加上 `./`，以确保该链接会被识别为相对路径即可。

但手动的一个个改有点过于麻烦了，所以我写了个正则表达式，利用 Linter 插件，实现“在按 `Ctrl+S` 保存时，自动在所有相对链接地址前加上 `./`。

### 基础设置

想要让 ob 的链接自动补全为相对链接的样式，首先要将设置改为下图红框中的样子：

![](https://cdn.pkmer.cn/images/202402271703835.png!pkmer)

正式使用时，直接输入 `[[` 即可自动补全。

### Linter

这是个用于自动格式化的插件^[格式化：自动调整 Markdown 语法，使其更加规范。]，需要将设置改为下图红框中的样子：

![](https://cdn.pkmer.cn/images/202402271703836.png!pkmer)

其中“相对路径修复”下方的三个框分别填上：

1. `(?<=([^\\]|^)\[(([^\]\r\n]|\\\])*[^\\\]\r\n])?(\\\\)*\]\()\/?(?<link>([^\.\\\)\/\:\#\r\n\s]|\.[^\.\\\)\/\:\r\n\s]|\.\.[^\\\)\/\:\r\n\s]|\\[^\\\/\:\r\n\s])(([^\)\:\r\n\s]|\\\))*[^\\\)\:\r\n\s])?(\\\\)*|\.{1,2})(?=\))`
2. `gm`
3. `./$<link>`

Obsidian 在解析路径时，绝对路径比相对路径优先级要高，要在相对链接的路径前加上 `./` 在能保证链接正确指向对应文件。

> [!warning]+ 注意！！！
>
> 我不能完全确保该正则表达式不会误编辑，因此强烈建议在使用 Git 或其它版本管理、备份工具的前提下使用该正则表达式，否则请尽可能手动添加 `./`。

## Update Relative Links

下载安装好该插件后，不需要设置，它是自动运行的。

Obsidian 的相对链接功能其实并不完整。当 A 文件中链接了 B 文件时，移动 B 文件的位置，链接的地址会自动更新为新的 B 文件地址。但移动 A 文件时，链接却不会自动更新，进而导致该链接失效。

本插件会自动在移动 A 文件时更新链接地址，确保链接的正确和有效。

同时为了保证该插件与链接地址的 `./` 格式更兼容，插件作者还给了 [插件代码的修改方法](https://github.com/val3344/obsidian-update-relative-links/issues/6)。如果读者用了上方的 Linter 插件与正则表达式，一定要记得按照插件作者说的修改一下插件。

## 闲言碎语

Obsidian 的相对链接用起来是真的麻烦，我之前整了好半天才把问题搞定，就算是这样也依旧不能保证正则表达式不会误编辑^[天知道那么多插件，会不会有什么特殊语法正好会被这个正则误伤。就算没有特殊语法，代码块中的内容也依然有概率被误伤。]。所以如果感觉太麻烦那就算了吧，用绝对链接其实也不是不行。