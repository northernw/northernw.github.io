---
title: 循序渐进Linux（5）-软件安装与管理
tags:
  - linux
  - 读书笔记
  - 循序渐进linux
categories:
  - 读书笔记
date: 2019-07-05 15:47:01
---
# 源码安装
- 下载解压 wget tar..
- ./configure, Makefile
- make
- make install

# rpm安装
- rpm -i[vh] file1.rpm file2.rpm 安装
- rpm -q[ag] package1...packageN 查询
- rpm -v[..] package1...验证
- -K: --checksig
- rpm -U[] file1.rpm .. fileN.rpm 更新
- rpm -e[] package1...N 删除 e=erase

# yum安装
yellowdog updater modified
- 使用rpm安装yum
- 配置yum：包括资源镜像列表等
- yum维护软件间的依赖性，可同时配置多个资源库，配置简单明了，保持与RPM数据库的一致性

## yum的基本用法
- 安装 yum install xx
- 更新
	- yum check-update
	- yum update
	- yum update kernel kernel-source
	- yum upgrade
	- ...
- 查询
	- yum info
	- yum info vsftpd
	- yum info perl* 可使用通配符
	- yum info updates
	- yum info installed
	- yum list updates
	- yum list
	- yum list gcc*
	- ...
- 不错的yum源
	- EPEL(Extra Packages for Enterprise Linux，企业版linux附加包)，epel-release
	- RPMForge, rpmforge-release


# 二进制软件安装
一般直接解压，或执行setup install install.sh等命令或脚本
