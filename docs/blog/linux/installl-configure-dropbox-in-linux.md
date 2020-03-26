---
date: '2017-04-13 10:21:07'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- linux
- dropbox
title: Linux下安装Dropbox-国内篇
---

- 如果您想在 Linux 桌面上使用 Dropbox，请安装相应的程序包。如果未列出您的发行版，请选择“从源代码编译”。

 - Ubuntu (.deb) [64-bit](https://www.dropbox.com/download?dl=packages/ubuntu/dropbox_2015.10.28_amd64.deb) [32-bit](https://www.dropbox.com/download?dl=packages/ubuntu/dropbox_2015.10.28_amd64.deb)
 - Fedora (.rpm)	[64-bit](https://www.dropbox.com/download?dl=packages/fedora/nautilus-dropbox-2015.10.28-1.fedora.x86_64.rpm) [32-bit](https://www.dropbox.com/download?dl=packages/fedora/nautilus-dropbox-2015.10.28-1.fedora.i386.rpm)
 - Debian (.deb) [64-bit](https://www.dropbox.com/download?dl=packages/debian/dropbox_2015.10.28_amd64.deb) [32-bit](https://www.dropbox.com/download?dl=packages/debian/dropbox_2015.10.28_i386.deb)
 - [从源代码编译](https://www.dropbox.com/help/247)
        
    !!! 注意：这些程序包安装了一个开放源码的帮助应用。此应用的版本不像 Dropbox 主应用一样频繁地更改。这些程序包总会安装 Dropbox Linux 版的最新版本。

- 通过命令行安装无外设模式的 Dropbox

    Dropbox 守护程序可在所有 32 位与 64 位 Linux 服务器上正常运行。若要安装，请在 Linux 终端运行下列命令。

    - 32-bit:

            cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86" | tar xzf -

    - 64-bit:

            cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -

- 参考[Linux终端下翻墙教程](/blog/linux/set-socks5-proxy-in-linux)一文以突破GFW封锁，俗称翻墙

- 接着，从新建的 .dropbox-dist 文件夹运行 Dropbox 守护程序。

        set http_proxy=127.0.0.1:8118
        set https_proxy=127.0.0.1:8118
        ~/.dropbox-dist/dropboxd
    
  !!! 如果是首次在服务器上运行 Dropbox，系统会要求您将链接复制并粘贴到运行的浏览器中，以便创建一个新的帐户或将服务器附加到现有帐户上。操作完成后，系统会在您的主目录中创建 Dropbox 文件夹。下载这个[Python脚本](https://www.dropbox.com/download?dl=packages/dropbox.py)，通过命令行控制 Dropbox。为了方便访问，请在 PATH 中的任何地方放入此脚本的符号链接。

- 实际上HTTP代理Privoxy只是在第一次Dropbox与服务器建立关联的时候需要，因为Dropbox本身支持socks5代理，后续可以使用上述的[Python脚本](https://www.dropbox.com/download?dl=packages/dropbox.py)来手动设置Dropbox的socks5代理来提高同步效率

        $ python dropbox.py proxy manual socks5 127.0.0.1 1080
        set