---
title: Spring Boot返回结果封装
date: 2020-04-08 00:28:17
tags: [Spring Boot]
categories: [Spring Boot]
---

创建统一的数据返回格式，有助于更好的同前端交互，也能让后端的代码看起来更早有呀简介。

<!--more-->

## 返回结果封装

Result.java。使用了Lombok。如果没有使用lombok，一定要重写toString()方法，不然返回调用Resut时将出错.

```java
@Data
public class Result<T> implements Serializable {
    private int code;
    private String msg;
    private T data;


    private Result(T data) {
        this.code = 200;
        this.data = data;
        this.msg = "操作成功";
    }

    private Result(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    /**
     * 成功直接返回数据和状态
     *
     * @param data 返回数据
     * @return
     */
    public static <T> Result<T> ok(T data) {
        return new Result<>(data);
    }

    public static <T> Result<T> ok() {
        return new Result<>(CodeMsg.SUCCESS.getCode(), CodeMsg.SUCCESS.getMsg());
    }


    /**
     * 失败调用
     */
    public static <T> Result<T> error(int code, String msg) {
        return new Result<>(code, msg);
    }
}

```

## 返回消息统一定义文件

```java
public enum CodeMsg {
    /**
     * 操作成功调用
     */
    SUCCESS(0, "成功"),
    ACTION_ERROR(11, "操作失败");


    private int code;
    private String msg;

    CodeMsg(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public int getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }
}
```

## 使用

在Controller层，如果要调用，可以这样（未在Service层处理）：

```java
@PostMapping("/")
public Result add(@RequestBody Note note) {
    log.info(note.toString());
    int res = noteMapper.insert(note);
    if (res == 1) {
        return Result.ok();
    } else {
        return Result.error(CodeMsg.ACTION_ERROR.getCode(), CodeMsg.ACTION_ERROR.getMsg());
    }
}
```

