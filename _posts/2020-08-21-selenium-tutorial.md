---
title: Selenium使用教程
date: 2020-08-21 14:57 +0800
categories: [Selenium]
tags: [selenium, crawler]
---
## 1.基础
Selenium是一个用于测试网站的自动化测试工具。Selenium测试直接运行在浏览器中，就像真正的用户在操作一样。
* 官方网站：<https://www.selenium.dev/>
* 官方文档：<https://www.selenium.dev/documentation/>
* API参考：<https://www.selenium.dev/selenium/docs/api/py/index.html>
* 用于测试的Web页面：<https://crossbrowsertesting.github.io/>
* 参考：<https://web.archive.org/web/20200810103659/http://www.testclass.net/selenium_python>

### 1.1 安装

```shell
pip install selenium
```

### 1.2 安装驱动
Selenium需要一个驱动来调用浏览器，主流浏览器驱动的下载链接如下：

| 浏览器 | 驱动下载链接 |
| --- | --- |
| Chrome | <https://chromedriver.chromium.org/downloads><br><https://chromedriver.storage.googleapis.com/index.html> |
| Edge | <https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/> |
| FireFox | <https://github.com/mozilla/geckodriver/releases> |
| Safari | <https://webkit.org/blog/6900/webdriver-support-in-safari-10/> |

驱动文件所在目录必须在PATH环境变量中。

以Chrome 84为例，下载驱动文件<https://chromedriver.storage.googleapis.com/84.0.4147.30/chromedriver_win32.zip>，将解压后的chromedriver.exe文件放在Python安装目录的Scripts目录下（该目录在安装Python时会被添加到PATH环境变量，在conda环境中对应环境下的Scripts目录会被添加到PATH环境变量的开头，从而优先被搜索）。

## 1.3 简单示例
（1）打开一个新的浏览器窗口，访问给定的URL

```python
from selenium import webdriver

driver = webdriver.Chrome()
driver.get('https://www.python.org/')
driver.quit()
```

![使用Selenium访问URL](/assets/images/selenium-tutorial/使用Selenium访问URL.png)

（2）使用百度搜索"selenium"

```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

with webdriver.Chrome() as driver:
    driver.get('https://www.baidu.com/')
    print(driver.title)
    search_box = driver.find_element_by_name('wd')
    search_box.send_keys('selenium' + Keys.ENTER)
```

（3）在单元测试中使用

```python
import unittest

from selenium import webdriver


class BaiduTests(unittest.TestCase):

    def setUp(self):
        options = webdriver.ChromeOptions()
        options.add_argument('--headless')  # 不显示浏览器窗口
        self.driver = webdriver.Chrome(options=options)

    def test_page_title(self):
        self.driver.get('https://www.baidu.com/')
        self.assertEqual('百度一下，你就知道', self.driver.title)

    def tearDown(self):
        self.driver.quit()


if __name__ == '__main__':
    unittest.main()
```

`ChromeOptions`参考：
* [Capabilities & ChromeOptions](https://web.archive.org/web/20200720021323/https://sites.google.com/a/chromium.org/chromedriver/capabilities/)（Recognized capabilities一节）
* [W3C WebDriver standard](https://www.w3.org/TR/webdriver/)
* [List of Chromium Command Line Switches](https://peter.sh/experiments/chromium-command-line-switches/)
* [selenium+python配置chrome浏览器的选项](https://blog.csdn.net/zwq912318834/article/details/78933910)
* [python+selenium+Chrome options参数](https://www.cnblogs.com/guapitomjoy/p/12150416.html)

## 2.WebDriver
官方文档：<https://www.selenium.dev/documentation/en/webdriver/browser_manipulation/>

Selenium通过驱动程序来调用浏览器，编程API为WebDriver，支持的浏览器及对应的类如下：

| 浏览器 | 类 |
| --- | --- |
| Chrome | `selenium.Chrome` |
| FireFox | `selenium.Firefox` |
| Edge | `selenium.Edge` |
| Internet Explorer | `selenium.Ie` |
| Safari | `selenium.Safari` |
| Opera | `selenium.Opera` |

创建浏览器

```python
from selenium import webdriver

driver = webdriver.Chrome()
# ...
driver.quit()
```

或

```python
from selenium import webdriver

with webdriver.Chrome() as driver:
    # ...
```

基本访问操作
* `get(url)`：访问URL
* `current_url`：当前URL
* `title`：当前页面的标题
* `back()`：后退
* `forward()`：前进
* `refresh()`：刷新

查找元素
* `find_element_by_xxx()`：查找一个匹配的元素
* `find_elements_by_xxx()`：查找所有匹配的元素

窗口（标签）操作
* `current_window_handle`：当前窗口句柄（标识窗口的字符串）
* `window_handles`：所有窗口的句柄（字符串列表）
* `switch_to_window(window)`：切换到句柄指定的窗口
* `minimize_window()`：最小化窗口
* `maximize_window()`：最大化窗口
* `fullscreen_window()`：全屏窗口
* `close()`：关闭当前窗口
* `quit()`：退出浏览器

Cookies操作
* `get_cookies()`：获得当前会话的所有Cookie，结果为字典列表，每个Cookie是一个字典
* `get_cookie(name)`：获得指定名称的Cookie，结果为一个字典，如果没有则返回`None`
* `delete_cookie(name)`：删除指定名称的Cookie
* `delete_all_cookies()`：删除当前会话的所有Cookie
* `add_cookie(cookie_dict)`：添加一个Cookie，`cookie_dict`是一个字典，必需的键为`"name"`和`"value"`，可选的键为`"path"`, `"domain"`, `"secure"`, `"expiry"`

截图
* `save_screenshot(filename)`：将当前窗口的截图保存为PNG文件

## 3.查找元素
WebDriver最基本的功能之一是在页面上查找元素（类似于[Scrapy的选择器]({% post_url 2020-08-16-scrapy-selectors %})）。

官方文档：<https://www.selenium.dev/documentation/en/getting_started_with_webdriver/locating_elements/>

`WebDriver`类和`WebElement`类提供了一组查找元素API
* `find_element_by_xxx()`：查找**一个**匹配的元素，返回`WebElement`对象，如果没有匹配的元素则产生`NoSuchElementException`异常
* `find_elements_by_xxx()`：查找**所有**匹配的元素，返回`WebElement`对象列表，如果没有匹配的元素则返回空列表

Selenium支持8中定位方式：

| 定位方式 | 方法 |
| --- | --- |
| id | `find_element_by_id()` |
| class属性 | `find_element_by_class_name()` |
| name属性 | `find_element_by_name()` |
| 标签名称 | `find_element_by_tag_name()` |
| 链接文本 | `find_element_by_link_text()` |
| 部分链接文本 | `find_element_by_partial_link_text()` |
| CSS选择器 | `find_element_by_css_selector()` |
| XPath | `find_element_by_xpath()` |

以上为查找一个元素的方法，查找所有元素的方法只需将`element`改为`elements`即可。

## 4.等待
`WebDriver.get()`方法会阻塞直到网页加载完成，有些网页的部分元素是通过JavaScript等方式异步加载的，此时在`get()`方法返回后立刻查找元素可能会失败。

`selenium.webdriver.support.ui.WebDriverWait`用于等待一段时间，直到指定的条件为真或超时为止。

官方文档：<https://www.selenium.dev/documentation/en/webdriver/waits/>

用法
* `WebDriverWait(driver, timeout)`：构造一个`WebDriverWait`对象，`driver`是`WebDriver`对象，`timeout`是超时时间，单位为秒
* `until(predicate)`：阻塞直到`predicate(driver)`为真或者超时，如果条件为真则返回`predicate(driver)`最后的返回值，如果超时则产生`TimeoutException`
* `until_not(predicate)`：阻塞直到`predicate(driver)`为假或者超时，如果条件为假则返回`predicate(driver)`最后的返回值，如果超时则产生`TimeoutException`

`selenium.webdriver.support.expected_conditions`模块提供了一些常用的判断条件，例如“标题等于”、“标题包含”、“元素存在”等。

例如，下面的代码将等待出现一个`<p>`元素，如果出现了`<p>`元素则`elem`被赋值为找到的第一个`<p>`元素，如果10秒后仍未出现则输出`"timeout"`。

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.common.exceptions import TimeoutException

driver.get('http://www.example.com/')
wait = WebDriverWait(driver, 10)
try:
    elem = wait.until(lambda d: d.find_element_by_tag_name('p'))
except TimeoutException:
    print('timeout')
```

## 5.WebElement
`WebElement`表示一个DOM元素，`WebDriver`和`WebElement`查找元素的方法均返回`WebElement`对象。

官方文档
* <https://www.selenium.dev/documentation/en/webdriver/web_element/>
* <https://www.selenium.dev/documentation/en/webdriver/keyboard/>
* <https://www.selenium.dev/documentation/en/support_packages/mouse_and_keyboard_actions_in_detail/>

获取元素信息
* `tag_name`：元素的标签名称
* `text`：元素的文本
* `get_attribute(name)`：获取元素的属性值
* `is_selected()`：返回单选框或复选框是否被选择
* `is_enabled()`：返回元素是否可用
* `is_displayed()`：返回元素是否对用户可见

查找元素
* `find_element_by_xxx()`：查找一个匹配的元素
* `find_elements_by_xxx()`：查找所有匹配的元素

键盘和鼠标操作
* `click()`：点击元素
* `send_keys(*value)`：模拟按键输入，`value`可以是字符串，也可以使用`selenium.webdriver.common.keys.Keys`中的常量
  * 例如：`send_keys('abc' + Keys.ENTER)`表示依次按A, B, C, Enter键，`send_keys(Keys.CONTROL, 'a')`表示Ctrl+A
* `submit()`：提交表单
* `clear()`：清空输入框文本 
* 更多操作：`ActionChains`，详见第6节

提示框：<https://www.selenium.dev/documentation/en/webdriver/js_alerts_prompts_and_confirmations/>

下拉框选择：<https://www.selenium.dev/documentation/en/support_packages/working_with_select_elements/>

截图
* `screenshot(filename)`：将当前元素的截图保存为PNG文件

## 6.ActionChains
`selenium.webdriver.ActionChains`类用于模拟键盘和鼠标操作。

官方文档：<https://www.selenium.dev/documentation/en/support_packages/mouse_and_keyboard_actions_in_detail/>

用法

```python
ActionChains(driver).action1().action2().perform()
```

其中`driver`是`WebDriver`对象，`action1()`, `action2()`表示`ActionChains`支持的操作，见下。

鼠标操作
* `click(element)`：点击元素（在元素上点击鼠标左键）
* `click_and_hold(element)`：在元素上按住鼠标左键
* `context_click(element)`：在元素上点击鼠标右键
* `double_click(element)`：双击元素
* `drag_and_drop(source, target)`：将元素`source`拖动到元素`target`
* `release(element)`：在元素上松开按住的鼠标键
* `move_to_element(element)`：将鼠标移动到元素中间

键盘操作
* `key_down(value, element)`：按下键
* `key_up(value, element)`：弹起键
  * 例如，按Ctrl+C：`ActionChains(driver).key_down(Keys.CONTROL).send_keys('c').key_up(Keys.CONTROL).perform()`
* `send_keys(*keys)`：向焦点元素发送按键，`keys`可使用字符串或`selenium.webdriver.common.keys.Keys`中的常量
* `send_keys_to_element(element, *keys)`：向元素发送按键（相当于`click()+send_keys()`）

延时
* `pause(seconds)`

## 7.坑
使用Selenium打开的浏览器`window.navigator.webdriver`是`true`，而正常的浏览器是`undefined`（可在浏览器Console中查看），因此网站根据这个特征很容易识别出是爬虫。

使用Selenium就无法登录LeetCode。

更多坑：<https://www.python66.com/seleniumjiaocheng/156.html>
