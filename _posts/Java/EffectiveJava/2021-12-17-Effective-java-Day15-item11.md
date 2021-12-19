---
title: '[Effective Java] Day 15 - Item 11 :: equals를 재정의하려거든 hashcode도 재정의하라.'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day15에서는 item 11에 대한 내용을 다룬다.

## Item 11 :: equals를 재정의하려거든 hashcode도 재정의하라.

그렇지 않으면 hash를 사용하는 HashMap, HashSet과 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.  