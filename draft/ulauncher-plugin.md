# 

## ulauncher 项目简介

> Ulauncher s a fast application launcher for Linux written in Python and uses GTK+ as a GUI toolkit.

项目地址: https://github.com/Ulauncher/Ulauncher

主页:https://ulauncher.io/

插件库地址:https://ext.ulauncher.io/

插件文档:http://docs.ulauncher.io/en/latest/

参考了MyIP(https://ext.ulauncher.io/-/github-rolfkoenders-ulauncher-myip),Run(https://ext.ulauncher.io/-/github-james-dumas-ulauncher-run)两个项目,MyIP主要参考了Ulauncher插件的基本写法,Run项目主要参考了怎么在点击子项之后新开调用自定义的函数

## 编写历程

首先是按照插件文档中写的,把demo跑起来,运行的时候文档上说manifest.json中的是`"manifest_version": "2",`,但是会报错,将版本改为1才能跑起来

然后编写辅助类List_choice,用来列出候选项,List_choice类有三个函数,check_exist(self,database_path)用于判断数据库文件database.txt(或者临时文件tmp)是否存在,不存在则新建一个;get_top_five(self,path)函数通过传入一个短路径,然后在数据库文件中读取包含有短路径的相关完整路径,选取其权重最高的前五个返回(少于五个则全部返回);write_path(self,path)函数通过传入一个完整路径,将其在数据库中的权重加一

## 函数解析
main.py

```
def __init__(self):
        super(JumpExtension, self).__init__()
        self.subscribe(KeywordQueryEvent, KeywordQueryEventListener())
        self.subscribe(ItemEnterEvent, ItemEnterEventListener())
```

初始化时要加上`self.subscribe(ItemEnterEvent, ItemEnterEventListener())`,因为通过点击子项会触发两段动作,更新数据库文件和打开文件,如果只是打开文件的话可以不加上

```
def on_event(self, event, extension):
        path = event.get_argument()
        print "path is %s " % path
        query_result = List_choice().get_top_five(path)
        items = []
        for i in range(len(query_result)):
            print "get result %d" % i
            print "it is %s" %query_result[i]
            items.append(ExtensionResultItem(icon='images/icon.png',
                                             name='jump to %s' % query_result[i][0],
                                             description='jump to here and open in file',
                                             on_enter=ExtensionCustomAction(query_result[i][0])))

        return RenderResultListAction(items)
```
关键在于on_enter之后触发的动作,由于是两段动作,所以需要自定义动作,然后import部分要加上`from ulauncher.api.shared.action.ExtensionCustomAction import ExtensionCustomAction`

然后我们需要新建一个ItemEnterEventListener继承类,在这里处理更新数据库文件和打开目录
```
class ItemEnterEventListener(EventListener):

    def on_event(self, event, extension):
        data = event.get_data() or ""
        print "data is %s " % data
        List_choice().write_path(data)
        print "before OpenAction"
        if sys.platform.startswith('darwin'):
            subprocess.call(('open', data))
        elif os.name == 'nt':
            os.startfile(data)
        elif os.name == 'posix':
            subprocess.call(('xdg-open', data))
        print "after OpenAction"
```
值得注意的是,这里并没有直接调用OpenAction()函数,因为调用了该函数却没起作用,不能打开文件,然后看了下import,`from ulauncher.api.shared.action.OpenAction import OpenAction`,然后在本地`/usr/lib/python2.7/dist-packages/ulauncher/api/shared/action`中找到对应的模块,
```
import sys
import os
import subprocess
from .BaseAction import BaseAction


class OpenAction(BaseAction):
    """
    Run platform specific command to open either file or directory

    :param str path: file or dir path
    """

    def __init__(self, path):
        self.path = path

    def keep_app_open(self):
        return False

    def run(self):
        if sys.platform.startswith('darwin'):
            subprocess.call(('open', self.path))
        elif os.name == 'nt':
            os.startfile(self.path)
        elif os.name == 'posix':
            subprocess.call(('xdg-open', self.path))
```
然后直接搬过来用,但是会提醒os没有startfile这个函数,所以注释掉了
