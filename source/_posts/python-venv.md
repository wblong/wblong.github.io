---
title: python venv
date: 2019-01-20 21:56:42
tags: python
categories:
- python
---
## 安装Anaconda
## 配置Anaconda 包源

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --set show_channel_urls yes
```

## 使用Anaconda
```
//列出安装包
conda list 
//列出python虚拟环境
conda env list
//创建一个python虚拟环境python3.5
conda create -n python3.5 python=3.5
//激活python3.5虚拟环境
activate python3.5
//为python3.5虚拟环境安装virtualenv环境
conda install -n python3.5 virtualenv
//解除虚拟环境
deactivate
//删除python3.5虚拟环境
conda remove -n python3.5 --all
conda remove --name python3.5 virtualenv
```
## 切换安装32位和64包

```
//切换到32位版本
set CONDA_FORCE_32BIT=1
//切换到64位版本
set CONDA_FORCE_32BIT=
// 查看环境信息
conda info

```
## python包管理

> 基本原则是使用Anconda管理多个版本的python环境，使用pip及virtualenv管理python环境的副本及三方库。

```
//anaconda 管理python多版本切换
conda create -n python3.5 python=3.5
active python3.5
pip install virtualenv
// 纯净python环境
virtualenv --no-site-packages venv 
/venv/Script/activate
//备份三方python包
pip freeze > requirement.txt
//批量安装python三方包
pip install -r requirement.txt
/venv/Script/deactivate
deactivate

```