---
layout: post
title: intellij-springboot-热启动设置
date: 2018-05-12
categories: intellij
---

在intellij中开发spring boot应用时，即使在pom.xml文件中加入了spring-boot-devtools依赖，在修改了源文件后，spring boot仍然不会自动重启。这种情况与intellij的设定相关，intellij默认在系统运行过程中修改源代码时是不会重新编绎的，所在spring boot检测不到有改动，所以就没有触发热启动<br>

只要在intellij里设置项目运行时编绎就可以使spring boot  devtools组件正常工作：
 1. 使用快捷键：`ctrl+shift+alt+/`，Registry,在弹出的对话框里找到：compiler.automake.allow.when.app.running 并打勾启动
 2. 使用快捷键：`ctrl+shift+a`,输入compiler 找到compiler设置，在弹出的对话框里找到build project automatically 勾上启动
 3. 重新启动即可