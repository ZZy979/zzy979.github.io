---
title: 【暑期实习面经】美团-大数据部-后端开发
date: 2021-05-08 13:20:43 +0800
categories: [面经]
tags: [interview]
---
## 一面
2021年3月30日  
50分钟  
大数据部，BI领域  
（面试官人超好！）

简历项目

Java  
1.HashMap的实现原理  
2.线程安全的类：ConcurrentHashMap  
3.volatile关键字  
4.乐观锁、悲观锁  
5.Java String的长度限制（不知道，字符串常量最大长度为65534）

数据库  
1.ACID  
2.隔离级别

编程
1.扑克牌顺子（力扣 剑指Offer 61）  
2.[二叉树根节点到叶子节点的所有路径和](https://www.nowcoder.com/practice/185a87cd29eb42049132aed873273e83?tpId=188&tags=&title=&diffculty=0&judgeStatus=0&rp=1&tab=answerKey)
```python
class Solution:
    def sumNumbers(self , root ):
        self.nums = []
        if not root:
            print(0)
        else:
            self.dfs(root, [])
            print(sum(self.nums))
    
    def dfs(self, root, path):
        path.append(str(root.val))
        if not root.left and not root.right:
            self.nums.append(int(''.join(path)))
        if root.left:
            self.dfs(root.left, path)
        if root.right:
            self.dfs(root.right, path)
        path.pop()
```
3.场景题：设计一个类，用于停车场管理系统  
最初版本：space表示空位，enter(charNum, time)进入停车场，exit(charNum, time)离开停车场，getSpace()获取剩余空位，fee(hours)计算停车费  
增加业务需求：  
（1）停车场有多个入口和出口：enter()和exit()增加参数入口/出口id  
（2）VIP收费便宜：增加FeeCalculator类，getVIPLevel(carNum)查询VIP等级，fee(level, hours)计算停车费  
（3）包月：FeeCalculator类增加register(carNum)方法，记录注册日期，之后的一个月内免费  
（4）记录车主信息，身份证、护照等：增加Card类表示使用的证件，enter()和exit()增加card参数
```java
class Parking {
    private int space;
    private Map<String, Date> enterTime;
    private Map<String, Integer> enterId;
    private Map<String, Card> enterCard;
    private FeeCalculator feeCalculator = new FeeCalculator();
    
    public Parking(int space) {
        this.space = space;
    }
    
    public int getSpace() {
        return space;
    }
    
    public void enter(String carNum, Date time, int id, Card card) {
        space--;
        enterTime.put(carNum, time);
        enterId.put(carNum, id);
        enterCard.put(carNum, card);
    }
    
    public double exit(String carNum, Date time, int id) {
        Date start = enterTime.get(carNum);
        int hours = (time - start) / 3600;
        space++;
        return fee(carNum, hours);
    }
    
    private double fee(String carNum, int hours) {
        return feeCalculator.fee(feeCalculator.getVIPLevel(carNum), hours);
    }
}

class Card {
    private String type;
    private String id;
    
    
}

class FeeCalculator {

    public int getVIPLevel(String carNum) {
        
    }
    
    public double fee(int level, int hour) {
        
    }
    
    public void register(String carNum) {
        
    }
}
```
反问

## 二面
2021年3月31日  
1小时

简历项目

算法题  
无序数组最长连续递增序列的长度
```
例如：[200, 202, 201, 1,2,4,3, 100,101,50]
四个递增序列分别为
50
100,101
200,201,202
1,2,3,4
return  1,2,3,4 -> 4
时间复杂度O(n)
```

散列表：
```java
import java.util.Scanner;
import java.util.Set;
import java.util.HashSet;

public class Main {
    public static void main(String[] args) {
        int[] nums = new int[]{1, 10, 2, 11, 5, 4, 3, 12, 6};
        System.out.println(maxSeqLength(nums));
    }
    
    public static int maxSeqLength(int[] nums) {
        if (nums.length == 0)
            return 0;
        Set<Integer> s = new HashSet<>();
        for (int x : nums)
            s.add(x);
        int ans = 0;
        for (int x : nums)
            if (s.contains(x)) {
                int seqLength = 1;
                s.remove(x);
                for (int i = x - 1; s.contains(i); --i) {
                    ++seqLength;
                    s.remove(i);
                }
                for (int i = x + 1; s.contains(i); ++i) {
                    ++seqLength;
                    s.remove(i);
                }
                ans = Math.max(ans, seqLength);
            }
        return ans;
    }
}
```
存在的问题：溢出

反问  
实习时间  
你希望去什么样的公司
