# 1. 变量

`lua` 中可以根据变量的作用域将其分为三类：
+ `global` ：除非显式声明为局部变量，否则都默认为全局变量
+ `local` ：局部变量
+ 表字段：这种特殊的变量可以是除了 `nil` 以外的所有类型，包括函数

## 1. 1 定义

```lua
local d, f = 5, 10 --声明局部变量 d, f
d, f = 5, 10       --声明全局变量 d, f
d, f = 10          --声明全局变量 d, f, 其中 f 的值为 nil
```
如果只是定义了但是没有初始化变量，则静态存储变量被隐式初始化为 `nil`

## 1.2 示例

```lua
-- 变量定义:
local a, b
-- 初始化
a = 10
b = 30
print("value of a:", a)
print("value of b:", b)
-- 交换变量的值
b, a = a, b
print("value of a:", a)
print("value of b:", b)
f = 70.0/3.0
print("value of f", f)
```
结果：
```
value of a:     10
value of b:     30
value of a:     30
value of b:     10
value of f      23.333333333333
```

## 1.3 左值与右值

`lua` 中也含有左值表达式与右值表达式：
+ 左值表达式：引用内存位置的表达式被称之为左值表达式。左值表达式既可以出现在赋值符号的左边也可以出现在赋值符号的右边。
+ 右值表达式：术语“右值”指存在内存某个位置的数据值。我们不能为右值表达式赋值，也就是说右值表达式只可能出现在赋值符号的右边，而不可能出现在赋值符号的左边。

```lua
g = 20  -- 变量属于左值表达式，合法
10 = 20 -- 非法
-- lua 中允许一个赋值语句出现多个左值表达式和右值表达式
g, l = 20, 30
```

# 2. 数据类型

`lua` 是动态类型语言，变量没用类型，值才有类型，以下是 `lua` 的值类型：
+ `nil` ：用于区分是否有数据，`nil` 表示没有数据
+ `boolean` ：布尔值
+ `number` ：数值，表示双精度浮点数
+ `string` ：字符串
+ `function` ：函数
+ `userdata` ：表示任意 `c` 数据
+ `thread` ：线程
+ `table` ：表，表示一般的数组，符号表，集合，记录，图，树等等，它还可以实现关联数组。它可以存储除了 `nil` 外的任何值

## 2.1 type函数

`lua` 提供一个 `type` 函数来使得我们知道变量的类型