---
title: PyCharm conda环境下Python Console报错
date: 2020-07-01 15:00:39 +0800
categories: [Python]
tags: [python, ide, conda]
---
错误信息：

```
D:\anaconda3\envs\zhitu\python.exe D:\PyCharm\plugins\python\helpers\pydev\pydevconsole.py --mode=client --port=6763
Traceback (most recent call last):
  File "D:\PyCharm\plugins\python\helpers\pydev\pydevconsole.py", line 5, in <module>
    from _pydev_comm.pydev_rpc import make_rpc_client, start_rpc_server, start_rpc_server_and_make_client
  File "D:\PyCharm\plugins\python\helpers\pydev\_pydev_comm\pydev_rpc.py", line 4, in <module>
    from _pydev_comm.pydev_server import TSingleThreadedServer
  File "D:\PyCharm\plugins\python\helpers\pydev\_pydev_comm\pydev_server.py", line 4, in <module>
    from _shaded_thriftpy.server import TServer
  File "D:\PyCharm\plugins\python\helpers\third_party\thriftpy\_shaded_thriftpy\server.py", line 9, in <module>
    from _shaded_thriftpy.transport import (
  File "D:\PyCharm\plugins\python\helpers\third_party\thriftpy\_shaded_thriftpy\transport\__init__.py", line 57, in <module>
    from .sslsocket import TSSLSocket, TSSLServerSocket  # noqa
  File "D:\PyCharm\plugins\python\helpers\third_party\thriftpy\_shaded_thriftpy\transport\sslsocket.py", line 7, in <module>
    import ssl
  File "D:\anaconda3\envs\zhitu\lib\ssl.py", line 98, in <module>
    import _ssl             # if we can't import it, let the error propagate
ImportError: DLL load failed while importing _ssl: 找不到指定的模块。
Process finished with exit code 1
```

解决方法：为解释器添加环境变量

打开PyCharm设置->Build, Execution, Deployment->Console->Python Console

在Environment variables输入框中输入PATH=D:\anaconda3\Library\bin，其中"D:\anaconda3"为Anaconda安装目录

![设置环境变量](/assets/images/resolve-pycharm-conda-python-console-error/设置环境变量.png)

点击确定，再次打开Python Console即可正常使用
