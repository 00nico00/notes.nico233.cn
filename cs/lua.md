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

# 6. 函数

## 6.1 定义

```lua
optional_function_scope function function_name( argument1, argument2, argument3..., argumentn)
function_body
return result_params_comma_separated
end
```
其中，函数也和变量一样默认全局作用域，可以使用 `local` 显式声明为局部作用域

## 6.2 变参函数

使用 `...` 作为参数可以创建参数个数可变的函数，即变参函数。
```lua
function average(...)
    local result = 0
    local arg = { ... }
    for i, v in ipairs(arg) do
        result = result + v
    end
    
    return result / #arg
end

print(average(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)) -- 5.5
```

# 7. 字符串

字符串初始化有三种方式：双引号，单引号，双中括号 `[[]]`
```lua
str1 = "123"
str2 = '456'
str3 = [[789]]
```

## 7.1 大小写操作函数

```lua
str1 = "Lua"
print(string.upper(str1))  -- LUA
print(string.lower(str1))  -- lua
```

## 7.2 替换子串

```lua
str = "123, world, 123"
new_str = string.gsub(str, "123", "hello")
print(new_str)  -- hello, world, hello
```

## 7.3 查找与颠倒

```lua
str = "hello, world"
print(string.find(str, "o, w"))    -- 5       8
reversed_str = string.reverse(str)
print(reversed_str)                -- dlrow ,olleh
```

## 7.2 格式化字符串

```lua
print(string.format("%s, %s", "hello", "world"))  -- 字符串格式化 hello, world
date = 2; month = 1; year = 2014
print(string.format("Date formatting %02d/%02d/%03d", date, month, year)) -- 日期格式化 Date formatting 02/01/2014
print(string.format("%.4f", 1 / 3))                                       -- 浮点数格式化 0.3333
```

# 8. 数组

在 `lua` 当中，数组是由整数索引表实现的，其大小并不是固定的，会根据元素的增加动态扩展大小。

## 8.1 一维数组

```lua
array = { "hello", "world" }
for i = 0, 2 do
    print(array[i])
end

-- result:
-- nil
-- hello
-- world
```

从以上例子可以看出，`lua` 访问一个不存在的索引时会得到 `nil` 值，而且其下标是默认从 1 开始的，不过也可以在索引小于或等于0的位置创造对象。

## 8.2 多维数组

创建多维数组主要有两种方式：
+ 创建数组的数组
+ 将多维数组映射到一位数组当中

```lua
array = {}
for i = 1, 3 do
    array[i] = {}
    for j = 1, 3 do
        array[i][j] = i * j
    end
end

for i = 1, 3 do
    for j = 1, 3 do
        print(array[i][j])
    end
end

--[[
1
2
3
2
4
6
3
6
9
]]
```

```lua
array = {}
for row = 1, 3 do
    for col = 1, 3 do
        array[row * 3 + col] = row * col
    end
end
  
for row = 1, 3 do
    for col = 1, 3 do
        print(array[row * 3 + col])
    end
end

--[[
1
2
3
2
4
6
3
6
9
]]
```

# 9. 迭代器

## 9.1 通用迭代器

通用迭代器可以访问集合中的键值对
```lua
array = { "hello", "world" }
for key, value in ipairs(array) do
    print(key, value)
end

--[[
1       hello
2       world
]]
```

在 `lua` 当中，使用函数表示迭代器，根据是否在函数中维护迭代器信息可以将迭代器分为两类：
+ 无状态迭代器
+ 有状态迭代器

## 9.2 无状态迭代器

此迭代器不会保存任何状态，例如：
```lua
function square(iteratorMaxCount, currentNumber)
    if currentNumber < iteratorMaxCount then
        currentNumber = currentNumber + 1
        return currentNumber, currentNumber * currentNumber
    end
end

for i, n in square, 3, 0 do
    print(i, n)
end

--[[
1       1
2       4
3       9
]]
```
这个迭代器用于输出 n 个数的平方值，我们将其封装一下，就能像 `ipairs` 那样调用了

```lua
function square(iteratorMaxCount, currentNumber)
    if currentNumber < iteratorMaxCount then
        currentNumber = currentNumber + 1
        return currentNumber, currentNumber * currentNumber
    end
end

  

function squares(iteratorMaxCount)
    return square, iteratorMaxCount, 0
end

for i, n in squares(3) do
    print(i, n)
end

--[[
1       1
2       4
3       9
]]
```

## 9.3 有状态迭代器

在 `lua` 中可以使用闭包来存储当前元素的状态。闭包通过函数调用得到变量的值。
```lua
function elementIterator(collection)
    local index = 0
    local count = #collection
    -- 返回闭包函数
    return function()
        index = index + 1
        if (index <= count) then
            -- 返回迭代器当前元素
            return collection[index]
        end
    end
end

array = { "hello", "world" }

for element in elementIterator(array) do
    print(element)
end

--[[
hello
world
]]
```

# 10. 表

`lua` 中，表是唯一可以用来创建不同数据类型的数据结构，比如常见的数组和字典。
```lua
-- 表初始化
mytable = {}

-- 表赋值
mytable[1]= "Lua"

-- 移除引用
mytable = nil
-- lua 的垃圾回收机制负责回收内存空间
```

注意，表的拷贝是**浅拷贝**，而且表的垃圾回收是**引用计数**的形式
```lua
a = { 1, 2, 3 }
b = a
a[1] = 4
for _, v in ipairs(b) do
    print(v)
end
a = nil
for _, v in ipairs(b) do
    print(v)
end

--[[
4
2
3
4
2
3
]]
```

