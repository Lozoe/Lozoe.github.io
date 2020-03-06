---
title: CSRF
date: 2020-03-05 11:50:38
categories: [安全]
tags:
---

## CSRF基本概念

CSRF，通常称为跨站请求伪造，英文名Cross-site request forgery

<!-- more -->

## CSRF攻击原理

![csrf攻击过程](csrf攻击过程.png)

可以被攻击的关键点：

1、在A网站登录

2、A网站存在漏洞接口

## CSRF防御

1、加token验证

2、reffer验证

3、隐藏令牌（隐藏token在http头）
