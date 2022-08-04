---
title: mybatis-plus
date: 2022-08-02 11:22:39
categories: Spring
tags: 
---

Mybatis-plus自带一个实体父类`Model`

该类的作用是能通过实体直接进行CRUD操作，不需要调用DAO，前提是必须存在对应的原始Mapper并继承BaseMapper并且可以使用的前提下。
