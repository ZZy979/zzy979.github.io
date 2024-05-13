---
title: Conda常用命令
date: 2019-10-19 15:57 +0800
categories: [Python]
tags: [python, conda]
---
下载Miniconda: <https://docs.conda.io/en/latest/miniconda.html>

创建环境：`conda create -n <env_name> [<pkg_name>...]`

激活环境：`conda activate <env_name>`

列出所有环境：`conda info -e`

列出已安装的包：`conda list`

安装包：`conda install <pkg_name>`

删除包：`conda remove <pkg_name>`

删除环境：`conda remove -n <env_name> --all`

安装包时报错：
PackagesNotFoundError: The following packages are not available from current channels

解决方法：添加channel

```shell
conda config --add channels conda-forge
```

（直接切换到对应环境下用pip安装即可）

指定Python版本：

```shell
conda create -n myenv python=3.8
```

（这里的"python"实际是一个conda包，使用conda list命令可以看到）

在Windows bat脚本中使用conda：<https://docs.conda.io/projects/conda/en/latest/user-guide/troubleshooting.html#using-conda-in-windows-batch-script-exits-early>

在Linux Shell脚本中使用conda：

```shell
source /path/to/anaconda3/etc/profile.d/conda.sh
conda activate <env>
```

参考：<https://github.com/conda/conda/issues/7980>
