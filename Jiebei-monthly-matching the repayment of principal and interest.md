---
title: Jiebei-monthly-matching the repayment of principal and interest
date: 2019-02-01 23:37:00
tags: 
---

﻿

﻿*注：花了半天研究了一下蚂蚁借呗的利息计算，起因是因为一次以为比较简单的计算发现比借呗上实际利息要少好几块钱，这就有点疑惑了。*

**说明：**
特别说明的只有两点

<!--more-->

 1. 利息分两部分：第一部分，距最近一个还款日期（即分期起息日，每个人不同，如27日）前的利息按日利率算，如今天24日，每月的还款日为27日，则24日到27日的三天累计日利息为第一部分利息；第二部分为分期起息日到最后一个还款日的分期总利息，按等额本息计算
 2.  第二部分利息等额本息计算的月利率为：日利率*（实际分期总天数）/3。而不是按30天算的月利率，比如第二部分5月27日到8月27日，实际天数为92天，实际计算的月利率为 0.00004\*(92/3)，即每月天数按实际总天数除以期数。
 附等额本息相关链接：[等额本息还款法](http://wiki.mbalib.com/wiki/%E7%AD%89%E9%A2%9D%E6%9C%AC%E6%81%AF%E8%BF%98%E6%AC%BE%E6%B3%95)


**举例：**
>本金为:1000 元,首个等额本息起息日为 27日，日利率为:0.0004 (首个还款日前的利息按日利率计算)
按30天计算月利率为：0.0120，实际月(期)利率=(几期实际天数/30)*日利率，为:0.012266666668
借款期数为：3期
等额本息起息日前的累计日利息(这部分按日利率算)为：1.20，首个等额本息起息日开始的几期总利息为：24.63302元
计算所得总利息为：25.83302 元

![实际总利息](https://img-blog.csdn.net/20180524101150520?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoZWV0YWhsb3Zlcg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**代码**：
```java
        //本金
        BigDecimal principal=new BigDecimal("1000");
        //日利率
        BigDecimal dayRate=new BigDecimal(4).divide(new BigDecimal("10000"));
        //还款日（分期起息日）
        Integer firstRepaymentDate=27;
        //分期起息日前天数（按日利率算）
        Integer days=3;
        //第一部分 累计日利息
        BigDecimal accumuDayInterest=new BigDecimal(days).multiply(dayRate).multiply(principal).setScale(2);
        //实际月利率
        double rate=dayRate.multiply(new BigDecimal("92").divide(new BigDecimal("3"),8,BigDecimal.ROUND_HALF_UP)).doubleValue();
        double month=3d;
        double monthRatePow=Math.pow(1+rate,month);
        BigDecimal dividerTop=principal.multiply(new BigDecimal(rate)).multiply(new BigDecimal(monthRatePow));
        BigDecimal dividerBottom=new BigDecimal(monthRatePow).subtract(new BigDecimal(1));
        BigDecimal monthResult=dividerTop.divide(dividerBottom,5,BigDecimal.ROUND_HALF_UP);
        //第二部分 累计月(期)利息
        BigDecimal monthlyInterest=monthResult.multiply(new BigDecimal(month)).subtract(principal);
        BigDecimal result=accumuDayInterest.add(monthlyInterest);
        System.out.println("本金为:"+principal+" 元,"+"首个等额本息起息日为 "+firstRepaymentDate+"日"+"，"+"日利率为:"+dayRate+" (首个还款日前的利息按日利率计算)");
        System.out.println("按30天计算月利率为："+new BigDecimal(30).multiply(dayRate)+"，"+"实际月(期)利率=(几期实际天数/30)*日利率，为:"+rate);
        System.out.println("借款期数为："+month+"期");
        System.out.println("等额本息起息日前的累计日利息(这部分按日利率算)为："+accumuDayInterest+"，"+"首个等额本息起息日开始的几期总利息为："+monthlyInterest+"元");
        System.out.println("计算所得总利息为："+result+" 元");
```
***