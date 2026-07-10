---
title: "Type Erasure Deep Dive"
date: 2026-04-22
draft: true
comments: true
categories: ["Github", "Blogging"]
tags: ["github-pages", "hugo", "setup"]
---

## What is Type Erasure ##

Consider the class

```java
public class Interval2<T extends Serializable & Comparable> implements Serializable {

    private T lower;
    private T upper;

    public Interval2(T first, T second) {
        if (first.compareTo(second) <= 0 ) {
            lower = first;
            upper = second;
        } else {
            lower = second;
            upper = first;
        }
    }
}
```

What happens if we change the order of the bounds i.e,

```<T extends Compalarable & Serialazable>``` to
```<T extends Serializable & Comparable>``` ?
