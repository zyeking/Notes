# Hello Manjaro

## 安装
+ refus
+ manjaro linux iso文件
+ U盘

## 基本配置
### 系统配置
添加国内源
```sh
sudo pacman-mirrors -c China
```
添加`archlinuxcn`中国源
```sh
sudo nvim /etc/pacman.conf
[archlinuxcn]
Server = https://mirrors.cloud.tencent.com/archlinuxcn/$arch
```
更新
```sh
sudo pacman -Syyu
```
安装yay以安装AUR包
```sh
sudo pacman -Sy yay
```
修改系统时间
```sh
sudo timedatectl set-local-rtc true
```

### 配置Github
+ 生成ssh-key, 添加`id_rsa.pub`到Github

```sh
ssh-keygen -t -rsa -C "your_email@example.com"
```

+ 设置用户名和邮件

```sh
git config --global user.name "your_name"
git config --global user.email "your_email"
git config -l # 查看配置
```

### 优化shell
```sh
sudo pacman -Sy fish # 安装fish
which fish # 查看fish位置
chsh -s /usr/bin/fish # 修改默认shell
fish_config # shell美化
```
### 优化终端
原生终端: 设置透明背景

simple terminal
```sh
git clone https://github.com/zyeking/st_config.git
```

### 配置Nvim
```sh
yay -Sy neovim # 安装
yay -Sy npm nodejs # 安装npm和nodejs以安装coc插件
npm config set registry https://registry.npm.taobao.org # 修改npm国内源
npm config get registry # 测试
sudo touch ~/.config/nvim/init.vim # 创建配置文件
cd ~/.config/nvim
git clone https://github.com/zyeking/nvim.git
```

### 配置Golang


### 软件
输入法
```sh
yay -Sy fcitx-sogoupinyin
yay -Sy fcitx-im
yay -Sy fcitx-configtool
yay -Sy fcitx-qt4

# sudo nvim ~/.xprofile
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"

yay -Sy compton # 解决输入法黑边问题, 新名字为picom
compton -b # 可添加到Autostart中
```
其他软件
```sh
yay -Sy google-chrome # 谷歌浏览器
yay -Sy netease-cloud-music # 网易云音乐
yay -Sy electronic-wechat # 微信
yay -Sy xmind # XMind
yay -Sy calibre # 图书管理工具
yay -Sy nutstore # 坚果云
yay -Sy kdenlive # 视频剪辑工具
yay -Sy flameshot # 截图软件, 配置全局快捷键
yay -Sy typora # Markdown编辑器
yay -Sy latte-dock # dock栏
```
wps配置
```sh
yay -Sy wps-office # -cn为中文版
sudo nvim /usr/bin/wps # 添加以下配置解决无法输入中文的问题
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

### KDE美化
+ Netspeed
+ Event Calendar
+ Active Window Control

## 其他问题
[解决Manjaro重启壁纸重设问题](https://blog.csdn.net/a00289/article/details/101656683)

[ifconfig指令找不到](https://unix.stackexchange.com/questions/145447/ifconfig-command-not-found)
