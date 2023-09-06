---
title: Go智障的时间格式化
date: 2021-06-11 15:40:52 +0800
categories: [Go]
tags: [golang, formatting]
---
`time.Time`类型的`Format()`方法将时间格式化为字符串，但该方法的参数必须用一个固定的参考时间`"2006-01-02 15:04:05"`

用其他时间会得到奇怪的结果

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t := time.Date(2021, 5, 25, 18, 12, 34, 0, time.Local)
	fmt.Println(t)
	fmt.Println(t.Format("2006-01-02 15:04:05"))
	fmt.Println(t.Format("06-January-2 Mon 03:04:05 PM"))
	fmt.Println(t.Format("2021-05-25 18:12:34"))
}
```

输出如下：

```
2021-05-25 18:12:34 +0800 CST
2021-05-25 18:12:34
21-May-25 Tue 06:12:34 PM
25255-34-2534 58:525:612
```

Go的格式与yyyy-MM-dd HH:mm:ss格式的对应关系：
<table>
<tr><td rowspan="2">年</td><td>2006=yyyy</td></tr>
<tr><td>06=yy</td></tr>
<tr><td rowspan="4">月</td><td>01=MM</td></tr>
<tr><td>1=M</td></tr>
<tr><td>Jan=MMM</td></tr>
<tr><td>January=MMMM</td></tr>
<tr><td rowspan="2">日</td><td>02=dd</td></tr>
<tr><td>2=d</td></tr>
<tr><td rowspan="2">星期</td><td>Mon=EEE</td></tr>
<tr><td>Monday=EEEE</td></tr>
<tr><td rowspan="3">小时</td><td>15=HH</td></tr>
<tr><td>03=KK</td></tr>
<tr><td>3=K</td></tr>
<tr><td rowspan="2">分钟</td><td>04=mm</td></tr>
<tr><td>4=m</td></tr>
<tr><td rowspan="2">秒</td><td>05=ss</td></tr>
<tr><td>5=s</td></tr>
<tr><td rowspan="2">上午/下午</td><td>PM=a</td></tr>
<tr><td>pm无对应</td></tr>
</table>

其他字符会被原样保留

在上面的示例中，`"2021"="{2}{02}{1}"="{不带前导0的日期}{带前导0的日期}{不带前导0的月份}"="25255"`
