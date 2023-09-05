---
title: 卸载pip包并卸载其依赖包
date: 2021-01-08 22:23:47 +0800
categories: [Python]
tags: [pip]
---
原创工具程序，卸载指定的pip包并递归卸载其依赖包

使用方法：将以下代码保存为pip_uninst_rec.py，执行`python pip_uninst_rec.py <pkg>`即可

```python
import argparse
import os
from collections import deque

import pip._internal.commands.show as show_cmd


def main():
    parser = argparse.ArgumentParser(description='卸载pip包，并卸载其依赖包')
    parser.add_argument('package', help='要卸载的包')
    args = parser.parse_args()

    q = deque()
    try:
        q.append(next(show_cmd.search_packages_info([args.package]))['name'])
    except StopIteration:
        return
    uninstalled = set()
    while q:
        pkg = q.popleft()
        pkg_info = next(show_cmd.search_packages_info([pkg]))
        os.system('pip uninstall -y ' + pkg)
        uninstalled.add(pkg)
        for dependency_info in show_cmd.search_packages_info(pkg_info['requires']):
            if not set(dependency_info['required_by']) - uninstalled:
                q.append(dependency_info['name'])


if __name__ == '__main__':
    main()
```
