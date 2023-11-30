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
```lua
print(type("This is a string")) -- string
local t = 10
print(type(5.8 * 10))           -- number
print(type(true))               -- boolean
print(type(print))              -- function
print(type(type))               -- function
print(type(nil))                -- nil
print(type(type(ABC)))          -- string
```

注意：变量在初始化或赋值前，都指向 `nil` 。`lua` 中空字符串和 0  在条件检查时都被当作 `true`

# 3. 操作符

## 3.1 算术运算操作符

此处只说明比较特殊的，不包括 `+` ,  `-` , `*` , `%`
```lua
local a, b = 10, 20
print(a ^ 2) -- 100.0 幂操作符
print(-a)    -- -10   一元减操作符用于取反
```

## 3.2 关系运算符

```lua
-- ~= ：就是 == 运算符的反面，相等为假，否则为真
print(10 ~= 20) -- false
```

## 3.3 逻辑运算符

`and` 代替了 `&&` ，`or` 代替了 `||` ，`not` 代替 `!`

## 4.4 其它操作符

```lua
-- 1 .. 用于连接两个字符串
print("hello".."world") -- helloworld

-- 2 # 返回 string 或者 table 的长度
print(#"hello") -- 5
```

# 4. 循环

## 4.1 while

```lua
while (condition) do
	statement(s)
end

-- eg:
local a = 1;
while a < 5 do
    print(a)
    a = a + 1
end
```

## 4.2 for

```lua
for init, max/min value, increment do
	statement(s)
end

-- eg:
for i = 1, 10, 1 do
    print(i)
end

for i = 10, 1, -1 do
    print(i)
end
```

以上两个例子等价于 `c++` 当中的：

```cpp
for (int i = 1; i <= 10; i++) {
	std::cout << i << std::endl;
}

for (int i = 10; i >= 1; i--) {
	std::cout << i << std::endl;
}
```

## 4.3 repeat...until

```lua
repeat
	statement(s)
until (condition)

-- eg: 其实就是相当于其他语言的 do while
local a = 1
repeat
    print(a)
    a = a - 1
until (a < 1)
```

# 5. if

在 `lua` 当中，所有**布尔真**和**非 `nil`** 都当为 `true` ，把所有**布尔假**和 `nil` 当作假，注意，**0 会被当作真。**

```lua
if (boolean_expression) then
	statement(s)
end
```