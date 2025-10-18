# Lab5-支付逻辑漏洞





**姓名：王骏**

**学号：22020007104**

**专业：2022级计算机科学与技术**

**日期：2025.10.18**



<div STYLE="page-break-after:always"></div>

# 目录

[TOC]



## 实验内容

通过该实验了解支付逻辑漏洞，掌握常见的支付漏洞原理以及漏洞检测利用和漏洞防护。



## 操作步骤

### 环境准备

访问 `http://10.1.1.23/demo/`

用账号 `tom` 密码 `123456` 登录进入系统

进入系统后，发现当前余额只有 5 元，尝试购买 1 本书籍 1 和 1 本书籍 2，发现购买不成功，余额不足。

打开 `BurpSuite` 工具，浏览器设置手动代理，本地 8080 端口

```json
127.0.0.1：8080
```

### 实验步骤一

输入两本书的购买数量各为1，开启intercept，然后点击购买，可以看到购买请求的数据包

![image-20251018201327449](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/202510182013480.png)

更改bill1和bill2的数值为1，这样只需要2元，如果成功篡改就会剩5-2 = 3元，点击forward

![image-20251018201545453](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/202510182015476.png)

可以看到这里成功篡改了请求。

### 实验步骤二

**没有对购买商品数量进行限制的支付逻辑漏洞**

访问 `http://10.1.1.23/destoon/`，注册账户

![image-20251018202257799](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/202510182022848.png)

用户名tom1

密码是123456

自动登陆后点击右上角站内信件，查看信件“充值卡”

![image-20251018202708599](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/202510182027647.png)

```json
卡号:
6666586599
密码：   
05650316
```

回到主页，点击右上角商务中心，点击充值，选择充值卡充值

![image-20251018203015333](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/202510182030377.png)

输入卡号密码，自动充值

![image-20251018203111745](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/202510182031767.png)

![image-20251018203123903](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/202510182031932.png)

充值成功后，直接访问首页 `http://10.1.1.23/destoon/`，点击团购，看到正在进行促销的苹果 6，点击参团->购买，来到订单确认页面，购买一个苹果 6 手机价格 66.66 元，填写收货手机号码。

![image-20251018203314404](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/202510182033453.png)

打开intercept开始抓包，点击提交订单。手机号码要填11位的，不然会提示错误。

![image-20251018203404701](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/202510182034739.png)

成功抓包后尝试把购买数量改成负数 -10，点击forward

![image-20251018203531319](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/202510182035362.png)

如果成功篡改，应该会增加666元。

![image-20251018203936893](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/202510182039924.png)

关闭intercept，自动返回，可以看到购买记录

![image-20251018204004334](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/202510182040363.png)

查看资金流水，可以看到多了666元

![image-20251018204031800](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/202510182040837.png)

查看网页源码，可以发现负数判定被注释了，所以没有处理非法数据。

<img src="https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/202510182043398.png" alt="image-20251018204336372" style="zoom:50%;" />

### 实验步骤三

**支付过程中对购买商品编号进行篡改**

打开浏览器访问 `http://10.1.1.23/yershop/` 进行注册登录

首先点击首页 `1F` 位置的苹果，价格为 299 元，观察商品详情页的 url 链接，链接中有 `id/42` 的字样，推测可能为商品的 id 编号





### 实验步骤四



### 实验步骤五



## 实验总结
