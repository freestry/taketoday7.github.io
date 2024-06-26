---
layout: post
title:  使用JMeter运行Selenium脚本
category: selenium
copyright: selenium
excerpt: Selenium
---

## 1. 概述

在本教程中，我们将介绍使用JMeter运行Selenium脚本。

## 2. JMeter与Selenium脚本

**JMeter为性能和负载测试提供了一个开源解决方案**。它也可以用于功能测试。
但是随着CSS、JS和HTML5等技术的进步，我们将越来越多的逻辑和行为下放到客户端。因此，更多的事情会增加浏览器的执行时间。这些事情包括：

+ 客户端Javascript执行-AJAX、JS模板等。
+ CSS变换 – 3D矩阵变换、动画等。
+ 3rd方插件-Facebook喜欢双击广告等。

反过来，这可能会影响网站或Web应用程序的整体性能。但是JMeter没有这样的矩阵来衡量这些感知性能。
由于JMeter不是真正的浏览器，JMeter也无法测量客户端渲染(如加载时间或页面再现)的用户体验。

像Selenium这样的Web驱动程序可以在客户端(在本例中为浏览器)自动执行和收集上面讨论的性能指标。
因此，虽然JMeter负载测试会给系统带来足够的负载，但JMeter WebDriver计划将从用户体验的角度获得响应时间和其他行为。

因此，除了测量性能之外，我们还可以在使用带有JMeter的WebDriver集时测量其他行为。所以让我们进一步讨论这个问题。

## 3. 先决条件

在使用JMeter运行Selenium 脚本之前，应满足以下先决条件：

+ 我们应该在系统中安装JMeter。
+ 接下来，我们应该使用JMeter插件管理器安装“Selenium/WebDriver”插件。
+ 在系统中下载gecko驱动程序/firefox驱动程序二进制文件。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/selenium-junit-testng)上获得。