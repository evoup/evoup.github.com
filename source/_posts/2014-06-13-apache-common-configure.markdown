---
layout: post
title: "apache common configure"
date: 2014-06-13 15:35
comments: true
categories: [java]
---

基于maven使用apache的common-configuration的过程中，遇到一个依赖小问题，记录下

<!-- more -->

关于这个包commons-configuration-1.6.jar,注意一定要有三个包才能使用，见下pom片段

```
<dependencies>

    <dependency>

        <groupId>commons-configuration</groupId>

        <artifactId>commons-configuration</artifactId>

        <version>1.8</version>

    </dependency>

    <dependency>

        <groupId>commons-beanutils</groupId>

        <artifactId>commons-beanutils</artifactId>

        <version>1.8.0</version>

    </dependency>

    <dependency>

        <groupId>commons-jxpath</groupId>

        <artifactId>commons-jxpath</artifactId>

        <version>1.3</version>

    </dependency>

</dependencies>
```