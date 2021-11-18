# Mac Study

[TOC]

## sync mouse with windows

https://synergy-project.org/nightly

* set the terminal colorful
```
durd@FelixDu ~$ cat ~/.bash_profile
export CLICOLOR=1
export PS1='\[\033[01;33m\]\u@\h\[\033[01;31m\] \W\$\[\033[00m\] '
set -o vi
```
## Create bootable USB stick from ISO in Mac OS X

### Prepare the USB stick
Check your USB stick and make a backup if there is any important data on it, as the next steps are going to delete everything on it.

To prepare the USb stick we are going to delete all the partitions on the stick and create an empty partition. To do this we need to know the device name of the USB stick. Open a terminal and execute the following command:
```
$ diskutil list
```

You will see a list of disks and partitions. The goal is to identify the USB stick in this output. Depending on your system configuration your output might look different from this one. This appears to show 3 physical discs but it does not. The /dev/disk1 is a virtual disk created because of the partition encryption (FileVault 2) I enabled in Mac OS X.
```
/dev/disk0
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *500.1 GB   disk0
   1:                        EFI                         209.7 MB   disk0s1
   2:          Apple_CoreStorage                         399.5 GB   disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
   5:                 Apple_Boot Boot OS X               134.2 MB   disk0s5
/dev/disk1
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                  Apple_HFS MacOSX                 *399.2 GB   disk1
/dev/disk2
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *2.0 GB     disk2
   1:       Microsoft Basic Data UNTITLED 1              2.0 GB     disk2s1
```

As shown in the output above, the connected USB stick is a small 2.0 GB drive with a FAT partition on it. We are now going to remove this partition in the next step. For the following steps we will need the name of the disk which in this case is “/dev/disk2”.

==With the following command the data on the disk (your USB stick) will be deleted!==

```
$ diskutil partitionDisk /dev/disk2 1 "Free Space" "unused" "100%"
```

With this command the USB stick was re-partitioned to have 1 partition without formatting and 100% of the size of the stick. If you check it again with “diskutil list” you will see the changes already, also the USB stick will no longer be shown in the Finder.

### Copy the image to the USB stick
Now we can copy the disk image we created to the USB stick. This is done via the dd(1) command. This command will copy the image to the disk (substitute the appropriate disk name for your USB stick here, as with the re-partitioning command):
```
$ dd if=destination_file.img.dmg of=/dev/disk2 bs=1m
```

The dd command does not show any output before it has finished the copy process, so be patient and wait for it to complete.
```
$ diskutil eject /dev/disk2
```

To eject the USB stick, use the above command. After this is done, the bootable USB stick is ready to be used.


## Recory
1. 关闭电脑等待10秒钟；
2. 将电源适配器连接到电源和电脑（如果尚未连接的话）；
3. 在内建键盘上，先按下（左侧）shift-control-option 键和电源按钮；
4. 三秒钟后同时松开所有键和电源按钮（MagSafe 电源适配器上的 LED 指示灯可能会更改状态或暂时关闭。）；
5. 按一下电源键以启动电脑，在启动音响起之前立刻按住 command-option-P-R 键，这四个键一直保持按住的状态；
6. 电脑会反复重启，伴随启动声响起，当听到第二声启动声的时候松开所有键。


## Iterm2 rzsz <BR>
https://github.com/mmastrac/iterm2-zmodem


## OSX Using Beyond Compare as git difftool
```
git config --global diff.tool bc3
git config --global merge.tool bc3
git config --global mergetool.bc3 trustExitCode true
```
Launch Beyond Compare, go to the Beyond Compare menu and run Install Command Line Tools.


### 剪切、拷贝、粘贴和其他常用快捷键
| 快捷键                   | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| Command-X                | 剪切：移除所选项并将其拷贝到剪贴板。                         |
| Command-C                | 将所选项拷贝到剪贴板。这同样适用于 Finder 中的文件。         |
| Command-V                | 将剪贴板的内容粘贴到当前文稿或 app 中。这同样适用于 Finder 中的文件。 |
| Command-Z                | 撤销前一个命令。随后您可以按 Command-Shift-Z 来重做，从而反向执行撤销命令。在某些 app 中，您可以撤销和重做多个命令。 |
| Command-A                | 全选各项。                                                   |
| Command-F                | 查找：打开“查找”窗口，或在文稿中查找项目。                   |
| Command-G                | 再次查找：查找之前所找到项目出现的下一个位置。要查找出现的上一个位置，请按 Command-Shift-G。 |
| Command-H                | 隐藏最前面的 app 的窗口。要查看最前面的 app 但隐藏所有其他 app，请按 Command-Option-H。 |
| Command-M                | 将最前面的窗口最小化至 Dock。要最小化最前面的 app 的所有窗口，请按 Command-Option-M。 |
| Command-N                | 新建：打开一个新文稿或窗口。                                 |
| Command-O                | 打开所选项，或打开一个对话框以选择要打开的文件。             |
| Command-P                | 打印当前文稿。                                               |
| Command-S                | 存储当前文稿。                                               |
| Command-W                | 关闭最前面的窗口。要关闭该 app 的所有窗口，请按 Command-Option-W。 |
| Command-Q                | 退出 app。                                                   |
| Command-Option-Esc       | 强制退出：选择要强制退出的 app。或者，按住 Command-Shift-Option-Esc 3 秒钟来仅强制最前面的 app 退出。 |
| Command–空格键           | Spotlight：显示或隐藏 Spotlight 搜索栏。要从 Finder 窗口执行 Spotlight 搜索，请按 Command–Option–空格键。如果您使用多个输入源以键入不同的语言，那么这些快捷键会更改输入源，而非显示 Spotlight。 |
| 空格键                   | 快速查看：使用快速查看预览所选项。                           |
| Command-Tab              | 切换 app：在打开的 app 中切换到下一个最近使用的 app。        |
| Command-Shift-波浪号 (~) | 切换窗口：切换到最前面的 app 的下一个最近使用的窗口。        |
| Command-Shift-3          | 屏幕快照：拍摄整个屏幕的屏幕快照。了解更多屏幕快照快捷键。   |
| Command-逗号 (,)         | 偏好设置：打开最前面的 app 的偏好设置。                      |

### 睡眠、注销和关机快捷键
| 快捷键                          | 描述                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| 电源按钮                        | 轻按可打开 Mac 或将 Mac 从睡眠状态唤醒。  当 Mac 处于唤醒状态时按住 1.5 秒钟会显示一个对话框，询问您是要重新启动、睡眠还是关机。按住 5 秒钟会强制 Mac 关机。 |
| Command–Control–电源按钮        | 强制 Mac 重新启动。                                          |
| Command–Option–电源按钮         | 将 Mac 置于睡眠状态。                                        |
| Shift–Control–电源按钮          | 将显示器置于睡眠状态。                                       |
| Command–Control–电源按钮        | 退出所有 app，然后重新启动 Mac。如果任何打开的文稿有未存储的更改，系统将询问您是否要存储这些更改。 |
| Command–Option–Control–电源按钮 | 退出所有 app，然后关闭 Mac。如果任何打开的文稿有未存储的更改，系统将询问您是否要存储这些更改。 |
| Command-Shift-Q                 | 注销您的 OS X 用户帐户。系统将提示您确认。                   |
| Command-Shift-Option-Q          | 立即注销您的 OS X 用户帐户，且系统不提示您确认。             |

### 文稿快捷键

| 快捷键                 | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| Command-B              | 以粗体显示所选文本，或者打开或关闭粗体显示功能。             |
| Command-I              | 以斜体显示所选文本，或者打开或关闭斜体显示功能。             |
| Command-U              | 对所选文本加下划线，或者打开或关闭加下划线功能。             |
| Command-T              | 显示或隐藏“字体”窗口。                                       |
| Command-D              | 从“打开”对话框或“存储”对话框中选择“桌面”文件夹。             |
| Command-Control-D      | 显示或隐藏所选字词的定义。                                   |
| Command-Shift-冒号 (:) | 显示“拼写和语法”窗口。                                       |
| Command-分号 (;)       | 查找文稿中拼写错误的字词。                                   |
| Option-Delete          | 删除插入点左边的字词。                                       |
| Control-H              | 删除插入点左边的字符。也可以使用 Delete 键。                 |
| Control-D              | 删除插入点右边的字符。也可以使用 Fn-Delete。                 |
| Fn-Delete              | 在没有向前删除  键的键盘上向前删除。也可以使用 Control-D。   |
| Control-K              | 删除插入点与行或段落末尾处之间的文本。                       |
| Command-Delete         | 在包含“删除”或“不存储”按钮的对话框中选择“删除”或“不存储”。   |
| Fn–上箭头              | 向上翻页：向上滚动一页。                                     |
| Fn–下箭头              | 向下翻页：向下滚动一页。                                     |
| Fn–左箭头              | 开头：滚动到文稿开头。                                       |
| Fn–右箭头              | 结尾：滚动到文稿末尾。                                       |
| Command–上箭头         | 将插入点移至文稿开头。                                       |
| Command–下箭头         | 将插入点移至文稿末尾。                                       |
| Command–左箭头         | 将插入点移至当前行的行首。                                   |
| Command–右箭头         | 将插入点移至当前行的行尾。                                   |
| Option–左箭头          | 将插入点移至上一字词的词首。                                 |
| Option–右箭头          | 将插入点移至下一字词的词尾。                                 |
| Command–Shift–上箭头   | 选中插入点与文稿开头之间的文本。                             |
| Command–Shift–下箭头   | 选中插入点与文稿末尾之间的文本。                             |
| Command–Shift–左箭头   | 选中插入点与当前行行首之间的文本。                           |
| Command–Shift–右箭头   | 选中插入点与当前行行尾之间的文本。                           |
| Shift–上箭头           | 将文本选择范围扩展到上一行相同水平位置的最近字符处。         |
| Shift–下箭头           | 将文本选择范围扩展到下一行相同水平位置的最近字符处。         |
| Shift–左箭头           | 将文本选择范围向左扩展一个字符。                             |
| Shift–右箭头           | 将文本选择范围向右扩展一个字符。                             |
| Shift–Option–上箭头    | 将文本选择范围扩展到当前段落的段首，再按一次则扩展到下一段落的段首。 |
| Shift–Option–下箭头    | 将文本选择范围扩展到当前段落的段尾，再按一次则扩展到下一段落的段尾。 |
| Shift–Option–左箭头    | 将文本选择范围扩展到当前字词的词首，再按一次则扩展到后一字词的词首。 |
| Shift–Option–右箭头    | 将文本选择范围扩展到当前字词的词尾，再按一次则扩展到后一字词的词尾。 |
| Control-A              | 移至行或段落的开头。                                         |
| Control-E              | 移至行或段落的末尾。                                         |
| Control-F              | 向前移动一个字符。                                           |
| Control-B              | 向后移动一个字符。                                           |
| Control-L              | 将光标或所选内容置于可见区域中央。                           |
| Control-P              | 上移一行。                                                   |
| Control-N              | 下移一行。                                                   |
| Control-O              | 在插入点后插入一行。                                         |
| Control-T              | 将插入点后面的字符与插入点前面的字符交换。                   |
| Command–左花括号 ({)   | 左对齐。                                                     |
| Command–右花括号 (})   | 右对齐。                                                     |
| Command–Shift–竖线     | 居中对齐。                                                   |
| Command-Option-F       | 前往搜索栏。                                                 |
| Command-Option-T       | 显示或隐藏 app 中的工具栏。                                  |
| Command-Option-C       | 拷贝样式：将所选项的格式设置拷贝到剪贴板。                   |
| Command-Option-V       | 粘贴样式：将拷贝的样式应用到所选项。                         |
| Command-Shift-Option-V | 粘贴并匹配样式：将周围内容的样式应用到粘贴在该内容中的项目。 |
| Command-Option-I       | 显示或隐藏检查器窗口。                                       |
| Command-Shift-P        | 页面设置：显示用于选择文稿设置的窗口。                       |
| Command-Shift-S        | 显示“存储为”对话框或复制当前文稿。                           |
| Command–Shift–减号 (-) | 缩小所选项。                                                 |
| Command–Shift–加号 (+) | 放大所选项。Command–等号 (=) 可执行相同的功能。              |
| Command–Shift–问号 (?) | 打开“帮助”菜单。                                             |

### Finder 快捷键
| 快捷键                      | 描述                                                       |
| --------------------------- | ---------------------------------------------------------- |
| Command-D                   | 复制所选文件。                                             |
| Command-E                   | 推出所选磁盘或宗卷。                                       |
| Command-F                   | 在 Finder 窗口中开始 Spotlight 搜索。                      |
| Command-I                   | 显示所选文件的“显示简介”窗口。                             |
| Command-Shift-C             | 打开“电脑”窗口。                                           |
| Command-Shift-D             | 打开“桌面”文件夹。                                         |
| Command-Shift-F             | 打开“我的所有文件”窗口。                                   |
| Command-Shift-G             | 打开“前往文件夹”窗口。                                     |
| Command-Shift-H             | 打开当前 OS X 用户帐户的“个人”文件夹。                     |
| Command-Shift-I             | 打开 iCloud Drive。                                        |
| Command-Shift-K             | 打开“网络”窗口。                                           |
| Command-Option-L            | 打开“下载”文件夹。                                         |
| Command-Shift-O             | 打开“文稿”文件夹。                                         |
| Command-Shift-R             | 打开“AirDrop”窗口。                                        |
| Command-Shift-U             | 打开“实用工具”文件夹。                                     |
| Command-Option-D            | 显示或隐藏 Dock。即使您未打开 Finder，此快捷键通常也有效。 |
| Command-Control-T           | 将所选项添加到边栏（OS X Mavericks 或更高版本）。          |
| Command-Option-P            | 隐藏或显示 Finder 窗口中的路径栏。                         |
| Command-Option-S            | 隐藏或显示 Finder 窗口中的边栏。                           |
| Command–斜线 (/)            | 隐藏或显示 Finder 窗口中的状态栏。                         |
| Command-J                   | 调出“显示”选项。                                           |
| Command-K                   | 打开“连接服务器”窗口。                                     |
| Command-L                   | 为所选项制作替身。                                         |
| Command-N                   | 打开一个新的 Finder 窗口。                                 |
| Command-Shift-N             | 新建文件夹。                                               |
| Command-Option-N            | 新建智能文件夹。                                           |
| Command-R                   | 显示所选替身的原始文件。                                   |
| Command-T                   | 在当前 Finder 窗口中打开单个标签时显示或隐藏标签栏。       |
| Command-Shift-T             | 显示或隐藏 Finder 标签。                                   |
| Command-Option-T            | 在当前 Finder 窗口中打开单个标签时显示或隐藏工具栏。       |
| Command-Option-V            | 移动：将剪贴板中的文件从其原始位置移动到当前位置。         |
| Command-Option-Y            | 显示所选文件的快速查看幻灯片显示。                         |
| Command-Y                   | 使用“快速查看”预览所选文件。                               |
| Command-1                   | 以图标方式显示 Finder 窗口中的项目。                       |
| Command-2                   | 以列表方式显示 Finder 窗口中的项目。                       |
| Command-3                   | 以分栏方式显示 Finder 窗口中的项目。                       |
| Command-4                   | 以 Cover Flow 方式显示 Finder 窗口中的项目。               |
| Command–左中括号 ([)        | 前往上一文件夹。                                           |
| Command–右中括号 (])        | 前往下一文件夹。                                           |
| Command–上箭头              | 打开包含当前文件夹的文件夹。                               |
| Command–Control–上箭头      | 在新窗口中打开包含当前文件夹的文件夹。                     |
| Command–下箭头              | 打开所选项。                                               |
| Command–Mission Control     | 显示桌面。即使您未打开 Finder，此快捷键也有效。            |
| Command–调高亮度            | 打开或关闭目标显示器模式。                                 |
| Command–调低亮度            | 当 Mac 连接到多个显示器时打开或关闭显示器镜像功能。        |
| 右箭头                      | 打开所选文件夹。此快捷键仅在列表视图中有效。               |
| 左箭头                      | 关闭所选文件夹。此快捷键仅在列表视图中有效。               |
| Option-连按                 | 在单独窗口中打开文件夹，并关闭当前窗口。                   |
| Command-连按                | 在单独标签或窗口中打开文件夹。                             |
| Command-Delete              | 将所选项移到废纸篓。                                       |
| Command-Shift-Delete        | 清倒废纸篓。                                               |
| Command-Shift-Option-Delete | 清倒废纸篓（不显示确认对话框）。                           |
| Command-Y                   | 使用“快速查看”预览文件。                                   |
| Option–调高亮度             | 打开“显示器”偏好设置。此快捷键可与任一亮度键搭配使用。     |
| Option–Mission Control      | 打开“Mission Control”偏好设置。                            |
| Option–调高音量             | 打开“声音”偏好设置。此快捷键可与任一音量键搭配使用。       |
| 拖移时按 Command 键         | 将拖移的项目移到其他宗卷或位置。拖移项目时指针会随之变化。 |
| 拖移时按 Option 键          | 拷贝拖移的项目。拖移项目时指针会随之变化。                 |
| 拖移时按 Command-Option     | 为拖移的项目制作替身。拖移项目时指针会随之变化。           |
| Option-点按伸缩三角形       | 打开所选文件夹内的所有文件夹。此快捷键仅在列表视图中有效。 |
| Command-点按窗口标题        | 查看包含当前文件夹的文件夹。                               |



| 在启动期间按住                      | 描述                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| Shift ⇧                             | 以安全模式启动。                                             |
| Option ⌥                            | 启动进入启动管理器。                                         |
| C                                   | 从可引导的 CD、DVD 或 USB 闪存驱动器（如 OS X 安装介质）启动。 |
| D                                   | 启动进入 Apple Hardware Test 或 Apple Diagnostics，具体取决于您正在使用的 Mac。 |
| Option-D                            | 通过互联网启动进入 Apple Hardware Test 或 Apple Diagnostics。 |
| N                                   | 从兼容的 NetBoot 服务器启动。                                |
| Option-N                            | 使用默认的启动映像从 NetBoot 服务器启动。                    |
| Command (⌘)-R                       | 从 OS X 恢复功能启动。                                       |
| Command-Option-R                    | 通过互联网从 OS X 恢复功能启动。                             |
| Command-Option-P-R                  | 重置 NVRAM。当再次听到启动声后，请松开这些键。               |
| Command-S                           | 以单用户模式启动。                                           |
| T                                   | 以目标磁盘模式启动。                                         |
| X                                   | 从 OS X 启动宗卷启动，否则 Mac 将从非 OS X 启动宗卷启动。    |
| Command-V                           | 以详细模式启动。                                             |
| 推出键 (⏏)、F12、鼠标键或触控板按钮 | 推出可移动介质，如光盘。                                     |



### Mac install cherrytree
1. get the cherrytree src
2. download and install pygtk https://pilotfiber.dl.sourceforge.net/project/macpkg/PyGTK/2.24.0/PyGTK.pkg
3. ./cherrytree

### Mac mount nfs
```
sudo mount_nfs -o resvport 192.168.50.1:/tmp/mnt/NEWIFI /Users/durd/Documents/mnt/
```


## Mac的brew修改国内源

### aliyun
```
# 替换brew.git
cd "$(brew --repo)"
git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git
# 替换homebrew-core.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git
# 刷新源
brew update
```