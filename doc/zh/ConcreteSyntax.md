语法示例
-------

本文档中所使用的符号说明:
- expr --->  expr    代表求值/规约到 JSON Normal Form
- expr ===>  expr    代表解糖/等价变换到另一个 Jix 表达式

----------------------------------

### JSON语法容错扩展

对象键可省略引号:

```js
{x: 1, y: 2}  --->  {"x": 1, "y": 2}
```

容忍冗余逗号:

```js
{x: 1, y: 2, }  --->  {"x": 1, "y": 2}

[1, 2, 3, ]  --->  [1, 2, 3]
```

在对象内换行时，逗号可省略:

```js
{
  x: 1
  y: 2
}
```

行内注释:

```js
{
  x: 1 #x的值
  y: 2 #y的值
}
```

### 表达式,变量

运算:

```js
(1 + 1)  --->  2
```

在对象和数组中使用运算:

```js
{x: 1, y: (1 + 1)}  --->  {"x": 1, "y": 2}

[1, 2, 3, (1 + 3)]  --->  [1, 2, 3, 4]
```

变量（let绑定）:

```js
a = 1; (a)  --->  1

a = 1; b = 2; (a + b)  --->  3
```

在对象值中引用变量:

```js
a = 1;
b = 2;
{x: (a), y: (b)}  --->  {"x": 1, "y": 2}
```

在对象键中引用变量:

```js
a = "x";
b = "y";
{(a): 1, (b): 2}  --->  {"x": 1, "y": 2}
```

在对象值中引用对象内的其他字段 (被引用字段需用=而不是:定义):

```js
{
    pname = "hello";
    version = "0.1.0";
    name: (pname ++ "-" ++ version)
}
===>
let
    pname = "hello";
    version = "0.1.0";
in
{
    pname: (pname)
    version: (version)
    name: (pname ++ "-" ++ version)
}
```

```js
{
    host = "127.0.0.1";
    port = "80";
    url: ("http://$(host):$(port)")
}
```

### 文本字面量

在文本（对象值）中内嵌变量 (f-string):

```js
name = "xyz";
{say: (f"hello, $(name)")}  --->  {"say": "hello, xyz"}
```

在文本（对象键）中内嵌变量 (f-string):

```js
id = 123;
{(f"No.$(id)"): 99}  --->  {"No.123": 99}
```

多行文本 (语法类似python 三引号，但解析逻辑类似 nix 会消除公共缩进)

```js
txt = """
    hello
    this is a
    long text
"""
---> "hello\nthis is a\nlong text"

```

在多行文本中内嵌变量 (f-string):

```js
name = "abc";
txt = f"""
    hello
    my name
    is $(name)
"""
```

转义字符

```js
txt = "hello\nabc";
txt  ---> "hello\nabc"
```

单引号文本，不支持转义字符

```js
txt = 'hello\nabc';
txt  ---> "hello\\nabc"
```

单引号多行字符串，完全原样录入

```js
txt = '''
    hello\n my name
    is \t$(name)
'''  --->  "hello\\n my name\nis \\t$(name)"
```

(高级功能) 显式 f-string，可以修改变量嵌入标志字符（默认 $，支持修改为 @,#,$,%,^,&,*):

```js
name = "abc";
txt = f@"""
    echo hello, @(name), today is $(datetime)
"""
```

(高级功能) Guest语言语法高亮支持:

```js
name = "abc";
txt = f@"""bash
    echo hello, @(name), today is $(datetime)
"""
```

小结

| 多行 | 可嵌入变量 | 支持转义字符 | 语法         |
| ---- | ---------- | ------------ | ------------ |
| No   | No         | Yes          | " ... "      |
| No   | No         | No           | ' ... '      |
| No   | Yes        | Yes          | f" ... "     |
| No   | Yes        | No           | f' ... '     |
| Yes  | No         | Yes          | """ ... """  |
| Yes  | No         | No           | ''' ... '''  |
| Yes  | Yes        | Yes          | f""" ... """ |
| Yes  | Yes        | No           | f''' ... ''' |

### 函数

```js
f = λa (a + 1);
(f 2)  --->  3

g = λa λb (a + b);
(g 1 2)  --->  3

h = λa λb (c = a + b; c);
(h 1 2)  --->  3
```

### 基本类型

```js
a_str_example : str = "hello";
a_bool_example : bool = true;
a_int_example : int = 1;
a_float_example : float = 1.414;
a_null_example : nil = null;
```

### 复合类型

定义对象类型(product type 的一种):

```js
Student : Type = *{name: String, age: Int};
s : Student = {name: "Alice", age: 18};
(s)  --->  {"name": "Alice", "age": 18}
```

对象类型中的缺省类型:

```js
Nixpkgs = *{
    haskellPackages: Dict Drv,
    ...: Drv
};
pkgs : Nixpkgs = ...;
```

定义元组类型(product type的另一种):

```js
Student : Type = *[String, Int];
s : Student = ["Alice", 18];
(s)  --->  ["Alice", 18]
```

字典类型:

```js
students : Dict Int = {"Alice": 18, "Bob": 19};
(students)  --->  {"Alice": 18, "Bob": 19}
```

列表类型:

```js
students : List String = ["Alice", "Bob"];
(students)  --->  ["Alice", "Bob"]
```

集合类型:

```js
students : Set String = ["Alice", "Bob"];
(students)  --->  ["Alice", "Bob"]

xs : Set Int = [1 2 3];
ys : Set Int = [3 2 1];
(xs == ys)  --->  true
```

### 函数类型

声明函数的返回值类型:

```js
f = λa λb (
    c: Int
        = a + b;
    c
);
(f 1 2)  --->  3

声明函数的参数和返回值类型:

g = λa: Int λb: Int (
    c: Int
        = a + b;
    c
);
(g 1 2)  --->  3

函数的返回值类型为对象类型:

h = λa: Int λb: Int (
    c: *{x: Int, y: Int}
        = {x: (a), y: (b)};
    c
);
(h 1 2)  --->  {x: 1, y: 2}

分别声明返回值对象中的每个字段类型:

u = λa: Int λb: Int (
    c: Int = a + 1;
    d: Int = b + 2;
    {x: (c), y: (d)}
);
(u 1 2)  --->  {x: 2, y: 4}
```

### 高阶类型特性

类型修饰(refinement):

```js
Score = Int ^{check: λx (x >= 0 and x <= 100)};
x : Score = 90;
(x)  --->  90
```

类型修饰, 使用多个判定函数以优化报错信息:

```js
Score = Int ^{
    checks: [
        {check: λx (x >= 0), message: "should be positive"},
        {check: λx (x <= 100), message: "should be less than 100"}
    ]
};
x : Score = 90;
(x)  --->  90
```

和类型 (Sum Type / Disjoint Union / Discriminated Union):

```js
Boolean : Type = "True" | "False"
```

递归类型 (μ):

```js
Nat : Type = μn ( "Zero" | *["Succ", n] )

// Nat ≜ μ n. 1 | n
```

泛型 (Λ):

```js
List : Type -> Type = Λa (μl ("Nil" | *["Cons", a, (l a)]));

// List ≜ Λ A. μ L. 1 | (A × L A)

```

### 合并

可合并类型:

```js
Score : Type = Int ^{
    check: λx (x >= 0 and x <= 100),
    merge2: λx λy (assume x == y; x)
};
x : Score = 90;
(x)  --->  90

MergeList : Type -> Type = Λa ( List a ^{merge: λxs (concat xs)} );
xs : MergeList Int = [1 2];
ys : MergeList Int = [3 4];
(x <+> y)  --->  [1 2 3 4]
```

默认合并逻辑 (所有类型都默认能合并，但合并结果不一定是成功的):

对于未定义合并策略的类型，如 Int 等基本类型, 默认合并策略是，assume所有赋值都相等，并返回其一，如果未赋值或不相等则合并失败

```js
Int ===> Int ^{merge2: λx λy (assume x == y; x)};
```

除非自定义合并策略

```js
SumInt ===> Int ^{merge2: λx λy (x + y), empty: 0};

MergeSet = Λa (Set a ^{merge: λxs (union xs)});
MergeDict = Λa (Dict a ^{merge: λxs (unionDict xs)});
```

### 类 Nix 语法糖

深度嵌套的对象赋值

```js
{
    a.x.u = 1;
}
===>
{
    a = {
        x = {
            u = 1;
        };
    };
}
```

混合多种风格的对象赋值情形

```js
cfg.a = 1;
cfg.b = 2;
cfg.c.x = 42;
cfg.c.y = {u = 1; v = "hello"; t.op = 3};
cfg.c.y.w = "haha";
cfg

===>

{
    a = 1;
    b = 2;
    c = {
        x = 42;
        y = {
            u = 1;
            v = "hello";
            t = {
                op = 3;
            }
            w = "haha";
        };
    };
};
```

### 错误处理

#### 函数实现错误

错误处理（assert关键字）:

```js
quicksort = λxs (
    ys = quicksort-impl xs;
    assert (ys == mergesort xs);
    ys
);
(quicksort [3,1,2])  --->  [1,2,3]
```

错误处理（error关键字）:

```js
quicksort = λxs (
    ys = quicksort-impl xs;
    if (ys /= mergesort xs)
    then error "the sort result is wrong"
    else ys
);
(quicksort [3,1,2])  --->  [1,2,3]
```

错误处理, impossible 关键字, 用于逻辑上不可达的分支: 如果通过静态分析能确认它确实不可达, 则可以优化掉; 若无法确认, 则需要生成对应的运行时报错代码

```js
(if x >= 0 then 1 elif x < 0 then -1 else impossible)
```

#### 不合理的函数调用

错误处理（assume关键字）:

```js
div = λa λb (
    assume (b /= 0);
    a // b
);
(div 4 2)  --->  2
```

错误处理（blame关键字）:

```js
div = λa λb (
    if b == 0 then blame "The divisor cannot be zero" else a // b
);
(div 4 2)  --->  2
```

#### 推诿 (需要跟踪调用栈)

错误处理（deflect关键字）:

```js
quicksort = λxs (
    ys = deflect (quicksort-impl xs);
    ys
);
(quicksort [1,3,2])  --->  [1,2,3]
```

### 运行时编译

(一般来说 partial evaluation 会自动处理优化，并不需要手动调用 opt，但提供该内置函数可以让我们在 REPL 中调用它，以帮助理解优化过程)

opt 关键字用于AOT计算:

```js
x = opt (fib 20); x
```

opt 关键字用于JIT编译:

```js
f = λa λb (fib a + b);
g = opt (f 2)   ===>   g = λb (fib 2 + b)

f = λa λb (a * a + b);
g = opt (f 2)   ===>   g = λb (4 + b);
```

复杂一点的例子 (不太确定能不能自动优化，也许需要手动重写为let形式)

```js
for (range 10) (λn (for (range 100) (λm (opt (f n) m))))
===>  for (range 10) (λn (_g = (f n); for (range 100) (λm (_g m))))
```

### 委派(dispatch) / typeclass / adhoc polymorphism (待定)

*这块并没有想好，暂不实现，仅用作思路记录，目前忽略这部分文档*

一种简单的思路是类似动态 overload ，然后利用 partial eval 消除动态开销

```js

div = λ(a: Int) λ(b: Int) (c: Int = a // b; c);
div = λ(a: Float) λ(b: Float) (c: Float = a // b; c);
...

===>

div : λ(a: Int) λ(b: Int) (c: Int; c) | λ(a: Float) λ(b: Float) (c: Float; c);
div = λa λb (if (typefit a Int and typefit b Int) then (a // b) elif (typefit a Float and typefit b Float) then (a / b) else impossible)
...
```

另一种思路是传递类型参数 及 Evidence Dictionary Passing，然后同样可以利用 partial eval 消除动态开销
