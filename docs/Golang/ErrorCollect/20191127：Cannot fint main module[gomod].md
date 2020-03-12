# 20191127：Cannot fint the main module[gomod]

## 环境
1. 在尝试vscode的debug中输出了`go: cannot find main module; see 'go help modules'`

## 原因 & 解决
1. 根目录下没有`go.mod`文件, 需要`go mod init`初始化建立相关`.mod`文件;或者将环境变量中的`GO111MODULE=AUTO/OFF`
