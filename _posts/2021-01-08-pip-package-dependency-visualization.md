---
title: pip包依赖关系可视化
date: 2021-01-08 22:29:54 +0800
categories: [Python]
tags: [pip]
---
原创工具程序，使用Graphviz将pip包的依赖关系图可视化，效果如下图所示

![效果图](/assets/images/pip-package-dependency-visualization/效果图.png)

其中红色表示没有入边的顶点

代码如下

```python
import subprocess

import pip._internal.commands.list as list_cmd
import pip._internal.commands.show as show_cmd


def run():
    with open('pip_libs.dot', 'w', encoding='utf8') as f:
        f.write('digraph G {\n')
        packages = list_cmd.get_installed_distributions()
        pkg_infos = list(show_cmd.search_packages_info([p.project_name for p in packages]))
        for info in pkg_infos:
            if not info['required_by']:
                f.write('"{}" [color="red"];\n'.format(info['name']))
            else:
                f.write('"{}";\n'.format(info['name']))
        for info in pkg_infos:
            for p in info['required_by']:
                f.write('"{}" -> "{}";\n'.format(p, info['name']))
        f.write('}\n')
    try:
        subprocess.run(['dot', '-Tpng', 'pip_libs.dot', '-o', 'pip_libs.png'])
        print('关系图已输出到pip_libs.png')
    except FileNotFoundError:
        print('未安装Graphviz，关系图已输出到pip_libs.dot')


if __name__ == '__main__':
    run()
```
