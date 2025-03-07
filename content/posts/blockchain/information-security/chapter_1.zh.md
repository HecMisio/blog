---
layout: post
title: "信息安全简史"
description: "简单叙述信息安全的发展历史"
date: 2025-01-28T15:03:39+08:00
image: "/posts/blockchain/information-security/images/chapter_1-cover.jpg"
tags: ["Blockchain"]
---
<i>信息安全（Information Security）</i>的概念可以追溯到古代，人们为了保护信息的机密性、完整性和可用性而采取的各种方法和技术。这些方法和技术主要依赖于手工操作、简单的数学工具和物理手段。而现代信息安全主要与计算机和通信技术的发展密切相关。

## 古典信息安全
最早的密码学应用可以追溯到公元前，例如在古罗马时期，被应用于军事领域的<i>凯撒密码（Caesar Cipher）</i>，就是一种简单的<b>替换密码</b>。凯撒密码得名于罗马共和国时期的军事领袖<a href="https://zh.wikipedia.org/wiki/%E5%B0%A4%E5%88%A9%E7%83%8F%E6%96%AF%C2%B7%E5%87%B1%E6%92%92" target="_blank">尤利乌斯·凯撒（Julius Caesar）</a>，其加密算法是将明文中的所有字母都在字母表上向前或向后按照固定数目进行偏移。

<div align="center">

![Caesar cipher](https://upload.wikimedia.org/wikipedia/commons/thumb/2/2b/Caesar3.svg/640px-Caesar3.svg.png)
*凯撒密码*

</div>

使用数学方法对凯撒密码进行描述

<p>
加密
<math display="block"> <msub> <mrow> <mi> E </mi> </mrow> <mrow> <mi> n </mi> </mrow> </msub> <mrow> <mo> ( </mo> <mi> x </mi> <mo> ) </mo> </mrow> <mo> = </mo> <mrow> <mo> ( </mo> <mi> x </mi> <mo> + </mo> <mi> n </mi> <mo> ) </mo> </mrow> <mo> mod </mo> <mn> 26 </mn> 
</math>
解密
<math display="block"> <msub> <mrow> <mi> D </mi> </mrow> <mrow> <mi> n </mi> </mrow> </msub> <mrow> <mo> ( </mo> <mi> x </mi> <mo> ) </mo> </mrow> <mo> = </mo> <mrow> <mo> ( </mo> <mi> x </mi> <mspace width="4px"/> <mo> - </mo> <mspace width="4px"/> <mi> n </mi> <mo> ) </mo> </mrow> <mo> mod </mo> <mn> 26 </mn> 
</math>
其中 <i>n</i> 表示偏移量
</p>

此外在世界范围内的许多璀璨的古老文明中，都充斥着信息安全的身影。古希腊的斯巴达密码棒、古埃及的象形文字、古代中国的藏头诗、拆字法等，这些来自于不同文明的信息保护机制，涉及到人类社会生活的方方面面，尤其是在军事、政治、商业和文化等重要领域，都发挥了不可或缺的作用。

## 近代信息安全

<div align="center">

![Enigma](https://upload.wikimedia.org/wikipedia/commons/thumb/2/27/Enigma-plugboard.jpg/560px-Enigma-plugboard.jpg)
*恩尼格玛密码机*

</div>

密码学在近代军事战争中，也有着非常重要的应用。二战时期，纳粹德国使用了一种名为<i>恩尼格码（Enigma）</i>的密码机来对德军战略信息进行加密，这是一种高级机械加密系统，采用对称加密算法中的流加密，使得同盟国难以获取德军情报，给西欧战场的盟军造成了巨大的困扰。1941年，<a href="https://zh.wikipedia.org/wiki/%E8%89%BE%E4%BC%A6%C2%B7%E5%9B%BE%E7%81%B5">艾伦·图灵（Alan Turing）</a>在布莱切利庄园成功制造出<i>“图灵甜点（Turing's Bombe，又称炸弹机）”</i>，同时利用德军的加密漏洞，实现了对德军加密情报的完全破译，普遍认为恩尼格玛密码机的破译，让欧洲战场的胜利提前了2年。

<div align="center">

![Turing's Bombe](https://upload.wikimedia.org/wikipedia/commons/thumb/7/7a/Wartime_picture_of_a_Bletchley_Park_Bombe.jpg/440px-Wartime_picture_of_a_Bletchley_Park_Bombe.jpg)
*图灵甜点(炸弹机)*

</div>