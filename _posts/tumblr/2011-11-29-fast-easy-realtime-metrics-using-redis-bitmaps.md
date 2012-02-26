---
layout: post
title: Fast, easy, realtime metrics using Redis bitmaps
tags: 
---
[Fast, easy, realtime metrics using Redis
bitmaps](http://blog.getspool.com/2011/11/29/fast-easy-realtime-metrics-using-
redis-bitmaps/)

> To count unique users that logged in today, we set up a bitmap where each
user is identified by an offset value. When a user visits a page or performs
an action, which warrants it to be counted, set the bit to 1 at the offset
representing user id.

Redis のビットデータ型を使って、ユーザのアクティビティをカウント、集計する手法。

ユーザIDのオフセットにしてフラグをたてて、アグリゲートはpopulation
count、複数のデータセットの集計もそれぞれをUnionして最終結果のビットからpopulation countでとれる。ビットのpopulation
countはSSEのハードウェア命令でもサポートされているから高速で、Macbook Proで 10億のデータに対して21ms
しかかからない。これは賢いなぁ。

