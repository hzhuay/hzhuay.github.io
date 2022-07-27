---
≈title: daily_report
date: 2022-06-20 14:55:28
tags:
---

# 实习记录

## 计划

该模块分为3个月的长期计划，1个月的中期计划和1周的短期计划。

第一个月

| 技术 / 事项                | 掌握程度                             | 预计用时                                                     |
| -------------------------- | ------------------------------------ | ------------------------------------------------------------ |
| Spring                     | 了解用法，能够写简易demo             | 1周，先学习Spring Framework，预计暂时不需要SpringMVC。根据第一个教程的目录，需要学习完事务。 |
| Spring Boot / Spring Cloud | 了解用法，能够写简易demo             | 1周，学习Spring Boot和Spring Cloud，学习程度可以略低于Spring，但是尽量能写出demo |
| 上述其他技术               | 暂时了解即可。部分技术已经有所了解。 | 在完成Spring学习目标之后进行，预计1周以内，再做模拟需求的期间先学习马上需要用到的。 |
| 做模拟需求                 |                                      | 预计共1-2周                                                  |

学习Spring暂定基于以下网络教程：https://www.w3cschool.cn/wkspring/pesy1icl.html，https://www.nhooo.com/spring/spring-tutorial.html，http://c.biancheng.net/spring/module.html。优先级从高到低，后者作为前者的补充。

第二个月：

- （可能）拉取项目代码学习
- 学习和复习Mysql和Redis的原理部分（较深入）
- MyBatis和MyBatis-Plus的使用方法。

第三个月：

- 了解Kafka / ZooKeeper等分布式技术
- 了解分布式技术在该项目中的应用
- 

## 日报

### 6月20日

实习入职第一天。

- [x] 入职手续
- [x]  熟悉办公软件
- [x] 了解未来要使用的技术栈
  - Java开发，Spring全家桶：Spring / Spring Cloud / Open Feign / MyBatis-Plus / gRPC
  - 数据库和文件存储：seaweedFS / cockroach / Mysql / Reids
  - 分布式：Kafka / ZooKeeper
  - 项目管理和部署：Maven / Docker / Docker-Compose

需要了解的：Kafka / ZooKeeper / Maven / Docker / Docker-Compose

需要熟悉的（会用）：数据库

需要重点掌握的：Spring全家桶

彬沈：大约两周的时间学习，然后完成一个模拟需求，再进入项目。

明日计划：学习Spring的基础部分（IoC，AoP等），根据教程可以创建demo项目。

### 6月21日

无论是创建还是销毁，实现`InitializingBean`, `DisposableBean`接口调用的方法都晚于XML配置的方法。一般不建议使用接口实现， 因为XML配置更具灵活性。

beans标签下指定**default-init-method** 和 **default-destroy-method** 可以设置所有bean的默认初始化和销毁行为。
