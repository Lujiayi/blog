---
title: gecco 中 @JSVar 的使用方法
date: 2022-03-22 16:23:06
tags:
---

## gecco 中 @JSVar 的使用方法

#### String、Boolean、Integer

```javascript
const name = 'xiao ming';
const male = true;
const age = 23;
```

```java
@JSVar(var = "name")
private String name;

@JSVar(var = "male")
private Boolean male;

@JSVar(var = "age")
private Integer age;

// getter and setter
```

#### Object

```javascript
const man = {
    name: 'xiao ming',
    age: 23
}
```

```java
@JSVar(var = "man", jsonpath = "$.name")
private String name;

@JSVar(var = "man", jsonpath = "$.age")
private Integer age;

// getter and setter
```

#### Array

```javascript
const manList = ['xiao ming', 'xiao wang', 'xiao li'];
```

```java
@JSVar(var = "manList" , jsonpath = "$[0]")
private String man1;

// getter and setter
```

#### Function

```javascript
const manFn = () => console.log('this is a function');
```
暂时不知道如何获取