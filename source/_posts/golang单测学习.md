---
title: golang单测学习
date: 2022-04-16 10:38:39
tags: golang
categories: golang
---
*该文主要参考博主李文周*

## 一、Go单元测试基础
Go测试文件命名必须为_test.go,测试函数必须为Test*。
*_test.go文件中有三种类型的函数，单元测试函数、基准测试函数和示例函数:
|类型|格式|作用
|:-|:-|:-
|测试函数|函数名前缀为Test|测试程序的一些逻辑行为是否正确|
|基准函数|函数名前缀为Benchmark|测试函数的性能|
|示例函数|函数名前缀为Example|为文档提供示例文档|

go test -v: 输出完整的测试结果
go test -run=*: -run参数对应一个正则表达式,只有函数名匹配上的测试函数才会被go test命令执行
go test -short: 不会执行耗时测试用例
go test -cover: 测试覆盖率

**跳过某些测试用例**
```go
func TestA(t *testing.T) {
    if testing.Short() {
        t.Skip("short模式下会跳过该测试用例")
    }
}
```

**子测试**
```go
func TestB(t *testing.T) {
    t.Run("case1", func(t *testing.T){...})
    t.Run("case1", func(t *testing.T){...})
    t.Run("case1", func(t *testing.T){...})
}
```

**表格驱动测试**
```go
func TestC(t *testing.T) {
    // 定义测试表格
    // 这里使用匿名结构体定义了若干个测试用例
    // 并且为每个测试用例设置了一个名称
    tests := []struct{
        name string
        input int
        output int
    }{
        {"case 1", 1, 2},
        {"case 2", 2, 3},
        {"case 3", 100, 101},
    }
    // 遍历测试用例
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) { // 使用t.Run()执行子测试
            got := Add(tt.input)
            if !reflect.DeepEqual(got, tt.output) {
                t.Errorf("expected:%#v, got:%#v", tt.output, got)
            }
        })
    }
}
```

**并行测试**
```go
func TestD(t *testing.T) {
    t.Parallel() // 将TLog标记为能够与其他测试并行运行
    // 定义测试表格
    // 这里使用匿名结构体定义了若干个测试用例
    // 并且为每个测试用例设置了一个名称
    tests := []struct{
        name string
        input int
        output int
    }{
        {"case 1", 1, 2},
        {"case 2", 2, 3},
        {"case 3", 100, 101},
    }
    // 遍历测试用例
    for _, tt := range tests {}
}
```

## 二、
## 三、
## 四、