---
title: 【暑期实习面经】快手-后端开发一面（凉）
date: 2021-03-29 16:15:51 +0800
categories: [面经]
tags: [interview]
---
2021年3月29日  
1小时

自我介绍

简历项目  
1.反爬机制（IP代理池，Selenium，其他不知道）  
2.Scrapy如何去重，布隆过滤器（不知道）  
3.Scrapy如何发请求，为什么比其他框架快（不知道）  
4.Scrapy中间队列（不知道）  
5.逆向工程，charles抓包工具（不知道）

Java  
1.集合框架有哪些  
Collection  
List - ArrayList, LinkedList  
Queue - Deque - ArrayDeque, LinkedList  
Set - HashSet  
2.HashMap原理  
3.线程池的核心参数及作用（理解有偏差）  
4.1000个任务，主线程创建10个线程，每个线程执行100个任务，最后等待执行完，如何实现  
给任务和线程编号
```java
import java.util.Scanner;
public class Main {
    public static void main(String[] args) {
        ThreadPool pool = new ThreadPool();
        for (int i = 0; i < 1000; ++i)
            pool.execute(new Task(i));
        pool.join();
    }
}

class ThreadPool {
    private volatile Queue<Task> queue;
    private Set<WorkThread> threads;
    
    public ThreadPool() {
        queue = new LinkedList<>();
        for (int i = 0; i < 10; ++i) {
            WorkThread thread = new WorkThread(queue, i, 100);
            threads.add(thread);
            thread.start();
        }
    }
    
    public void execute(Task task) {
        queue.offer(task);
    }
    
    public void join() {
        for (WorkThread thread : threads)
            thread.join();
    }
    
}

class WorkThread extends Thread {
    private volatile Queue<Task> queue;
    private int executed = 0;
    
    public WorkThread(Queue<Task> queue, int n, int total) {
        super(String.valueOf(n));
        this.queue = queue;
        this.total = total;
    }
    
    @Override
    public void run() {
        while (true) {
            synchronized (queue) {
                Task task = queue.poll();
                if (task.getNum() % 10 == Integer.parseInt(this.name)) {
                    runTask(task);
                    executed += 1;
                }
                else
                    queue.offer(task);
                if (executed == total)
                    break;
            }
        }
    }
    
}
```
5.conda launch?（不知道）  
6.Java加锁方式：Lock+Condition, synchronized  
synchronized原理：对象内部锁  
内部锁如何实现（不知道）

编程：数字转汉字  
例如21467→二万一千四百六十七
```java
import java.util.Scanner;
import java.util.List;
import java.util.ArrayList;

public class Main {
    private static final String[] digit = new String[]{"零", "一", "二", "三", "四", "五", "六", "七", "八", "九"};
    private static final String[] exp = new String[]{"", "万", "亿"};
    
    public static void main(String[] args) {
        int n = 21467;
        List<String> res = new ArrayList<>();
        int e = 0;
        while (n > 0) {
            res.add(convert(n % 10000) + exp[e]);
            n /= 10000;
            ++e;
        }
        StringBuilder builder = new StringBuilder();
        for (int i = res.size() - 1; i >= 0; --i)
            builder.append(res.get(i));
        System.out.println(builder.toString());
    }
    
    public static String convert(int n) {
        // n <= 9999
        int n1 = n / 1000, n2 = (n - n1 * 1000) / 100;
        int n3 = (n - n1 * 1000 - n2 * 100) / 10, n4 = n % 10;
        String res = "";
        if (n1 > 0)
            res += digit[n1] + "千";
        if (n2 > 0)
            res += digit[n2] + "百";
        if (n3 > 0)
            res += digit[n3] + "十";
        if (n4 > 0)
            res += digit[n4];
        if (res.isEmpty())
            res = "零";
        return res;
    }
}
```

评价：最后代码写得还可以，知识掌握不够深入
