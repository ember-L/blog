# Ubuntu环境美化

安装ubuntu:[Ubuntu-24.04安装教程超详细(2024)_ubuntu24.04-CSDN博客](https://blog.csdn.net/hack_18520/article/details/143466277)

## 安装软件

在这里会安装tweaks美化管理工具

```sh
sudo apt update
sudo apt install gnome-tweaks chrome-gnome-shell
sudo apt install gtk2-engines-murrine gtk2-engines-pixbuf 
sudo apt install sassc optipng inkscape libcanberra-gtk-module libglib2.0-dev libxml2-utils
```



## Gnome插件

默认使用的是 gnome 的桌面环境，本文的美化也是基于 gnome 桌面环境，美化 gnome 桌面环境少不了安装 gnome 插件，gnome 插件的网址是：[https://extensions.gnome.org](https://link.zhihu.com/?target=https%3A//extensions.gnome.org)

> 插件

- userthemes：设置用户主题
- netspeed：显示网速
- Compiz alike magic lamp effect：暴风吸入式窗口最小化
- Compiz windows effect：Q弹的窗口

- blur-my-shell：将顶部的plane、dock模糊化



![image-20250520104557369](https://ember-img.oss-cn-chengdu.aliyuncs.com/image-20250520104557369.png)

### 安装

1. 打开firefox打开网站[https://extensions.gnome.org](https://link.zhihu.com/?target=https%3A//extensions.gnome.org)
2. 搜索插件，点击安装即可

![image-20250520112516677](./assets/image-20250520112516677-1748747688740-5.png)

## 主题美化

在Github上有类Macos主题：https://github.com/vinceliuice/WhiteSur-gtk-theme

### 安装

> 安装主题

```sh
 #克隆主题仓库
 git clone https://github.com/vinceliuice/WhiteSur-gtk-theme.git --depth=1
 
 #运行以安装默认的 WhiteSur GTK 主题包
 ./install.sh            # install all available theme accents
 
 # 安装firefox主题
 ./tweaks.sh -f
 
 # 安装锁屏界面
 sudo ./tweaks.sh -g -b blank
```

> 安装图标

```sh
#克隆图标库
git clone https://github.com/vinceliuice/WhiteSur-icon-theme.git

#安装
cd WhiteSur-icon-theme
./install.sh -a #安装图标
./install.sh -b #安装panel
```



### 设置

> 设置主题

使用`win+a`打开主菜单，打开`tweaks`，根据安装的主题进行设置

![](https://ember-img.oss-cn-chengdu.aliyuncs.com/image-20250520112305752.png)

## Shell美化

教程：[Linux终端环境配置 | GeekHour](https://www.geekhour.net/2023/10/21/linux-terminal/)

### 配置终端环境

#### 3.1 安装依赖工具

首先我们需要安装一些必要的支持工具，包括wget、git、curl和vim等等，

```sh
sudo apt install wget git curl vim -y
```



3.2 安装zsh
连接成功之后就可以开始配置终端环境了，首先我们来把当前的shell切换成zsh，ubuntu系统默认的shell是bash，可以使用echo $SHELL命令来查看当前使用的shell，zsh是bash的一个替代品，它的功能更加强大和丰富，可以使用cat /etc/shells来查看支持的shell

```sh
$ cat /etc/shells
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
/usr/bin/zsh
/bin/zsh
/bin/ksh
/bin/rksh
/usr/bin/ksh
/usr/bin/rksh
/bin/csh
/bin/tcsh
/usr/bin/csh
/usr/bin/tcsh
```

如果结果中没有zsh的话就需要使用下面的命令来安装一下：

```
sudo apt install zsh -y

```



#### 3.3 安装字体

终端的一些iconfont需要一些特殊字体才能完美显示，推荐使用Nerd字体，官网：nerdfonts.com/
powerlevel10k主题推荐使用MesloLGS-Nerd字体，一般在初次安装配置主题的时候会默认提示安装，但是如果没有正常安装的话也可以使用下面的内容来手动安装一下：MesloLGS字体ttf文件下载地址：

```sh
wget https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf &&
wget https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf  &&
wget https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf  &&
wget https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf
```

或者Mac也可以使用Homebrew来安装

```sh
# Mac homebrew
brew tap homebrew/cask-fonts &&
brew install --cask font-<FONT NAME>-nerd-font

e.g.
brew tap homebrew/cask-fonts
brew install --cask font-code-new-roman-nerd-font
```




安装完成之后再系统设置或者各个软件比如终端或者VSCode上把字体设置为MesloLGS NF就可以了。

如果是没有安装KDE或者Gnome图形界面的Linux的话，可以使用下面的命令来设置一下：

```sh
# Linux安装字体

sudo cp ttf/*.ttf /usr/share/fonts/truetype/

# 安装fontconfig

sudo apt install fontconfig

# 刷新字体缓存

fc-cache -fv
```



#### 3.4 安装Oh-My-Zsh

执行下面的语句就可以安装了。

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

```

慢或者失败的小伙伴可以换成国内源:

```
wget https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh
```



下载之后给install.sh添加执行权限：

```
chmod +x install.sh

```

然后还需要修改一下远程仓库地址：
使用vim打开install.sh文件(vim install.sh)后，找到以下部分：

```sh
# Default settings

ZSH=${ZSH:-~/.oh-my-zsh}
REPO=${REPO:-ohmyzsh/ohmyzsh}
REMOTE=${REMOTE:-https://github.com/${REPO}.git}
BRANCH=${BRANCH:-master}
```

将中间两行修改为下面这样，使用gitee镜像：

```sh
REPO=${REPO:-mirrors/ohmyzsh}
REMOTE=${REMOTE:-https://gitee.com/${REPO}.git}
```



然后保存退出：`:wq`
再执行一下，一般就应该安装好了。

将系统默认shell切换为zsh

```sh
# 切换默认shell

chsh -s $(which zsh)

# 确认

echo $SHELL
```



#### 3.5 安装Zsh主题和插件

```sh
# powerlevel10k主题

git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k

# zsh-autosuggestions自动提示插件

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# zsh-syntax-highlighting语法高亮插件

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# 配置powerlevel10k

p10k configure
```




编辑~/.zshrc文件启用插件和主题

```sh
# 修改主题

ZSH_THEME="powerlevel10k/powerlevel10k"

# 启用插件

plugins=(
  git
  zsh-autosuggestions
  zsh-syntax-highlighting
)
```



