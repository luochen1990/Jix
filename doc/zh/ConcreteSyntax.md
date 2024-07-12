语法示例
-------

说明:
expr --->  expr    代表求值/规约
expr ===>  expr    代表解糖/等价变换

----------------------------------

### JSON语法容错扩展

对象键可省略引号:
> {x: 1, y: 2}  --->  {"x": 1, "y": 2}

容忍冗余逗号:
> {x: 1, y: 2, }  --->  {"x": 1, "y": 2}
> [1, 2, 3, ]  --->  [1, 2, 3]

对象内换行可替代逗号:

> {
>   x: 1
>   y: 2
> }

行内注释:

> {
>   x: 1 #x的值
>   y: 2 #y的值
> }

### 表达式,变量,函数

运算:
> (1 + 1)  --->  2

在对象和数组中使用运算:
> {x: 1, y: (1 + 1)}  --->  {"x": 1, "y": 2}
> [1, 2, 3, (1 + 3)]  --->  [1, 2, 3, 4]

变量（let绑定）:
> a = 1; (a)  --->  1
> a = 1; b = 2; (a + b)  --->  3

在对象值中引用变量:
> a = 1; b = 2; {x: (a), y: (b)}  --->  {"x": 1, "y": 2}

在对象键中引用变量:
> a = "x"; b = "y"; {(a): 1, (b): 2}  --->  {"x": 1, "y": 2}

在对象值中内嵌变量:
> name = "xyz"; {say: ("hello, $(name)")}  --->  {"say": "hello, xyz"}

在对象键中内嵌变量:
> id = 123; {("No.$(id)"): 99}  --->  {"No.123": 99}

在对象值中引用对象内的其他字段:
> { pname: "hello", version: "0.1.0", name: (pname <+> "-" <+> version) }
> { host: "127.0.0.1", port: "80", url: ("http://$(host):$(port)") }

函数:
> f = ?a (a + 1); (f 2)  --->  3
> f = ?a ?b (a + b); (f 1 2)  --->  3
> f = ?a ?b (c = a + b; c); (f 1 2)  --->  3

### 类型

带类型声明的函数:
> f = ?a ?b (c: Int = a + b; c); (f 1 2)  --->  3
> f = ?a: Int ?b: Int ( c: Int = a + b; c ); (f 1 2)  --->  3
> f = ?a: Int ?b: Int ( c: {x: Int, y: Int} = {x: a, y: b}; c ); (f 1 2)  --->  3
> f = ?a: Int ?b: Int ( c: Int = a + 1; d: Int = b + 2; {x: c, y: d} ); (f 1 2)  --->  3

定义对象类型:
> Student = @{name: String, age: Int}; s : Student = {name: "Alice", age: 18}; (s)  --->  {"name": "Alice", "age": 18}

对象类型中的缺省类型:
> Nixpkgs = @{haskellPackages: Dict Drv, ...: Drv};

定义元组类型:
> Student = @[String, Int]; s : Student = ["Alice", 18]; (s)  --->  ["Alice", 18]

字典类型:
> students : Dict Int = {"Alice": 18, "Bob": 19}; (students)  --->  {"Alice": 18, "Bob": 19}

列表类型:
> students : List String = ["Alice", "Bob"]; (students)  --->  ["Alice", "Bob"]

集合类型:
> students : Set String = ["Alice", "Bob"]; (students)  --->  ["Alice", "Bob"]
> xs : Set Int = [1 2 3]; ys : Set Int = [3 2 1]; (xs == ys)  --->  true

类型修饰(refinement):

> Score = Int ^{check: ?x (x >= 0 and x <= 100)}; x : Score = 90; (x)  --->  90
> Score = Int ^{
>     checks: [
>         {check: ?x (x >= 0), message: "should be positive"},
>         {check: ?x (x <= 100), message: "should be less than 100"}
>     ]
> };
> x : Score = 90; (x)  --->  90

对象类型中的计算字段:
> Student = @{
>   name: String
>   age: Int
>   id: Sha256 = hash name <+> hash age
> };
> s : Student = {name: "Alice", age: 18}; (s)
>  --->  {"name": "Alice", "age": 18, "id": "eropupuahasdkzxchoqnjdas"}

Sum Type (Disjoint/Discriminated Union):
> Bool = Yes | No

泛型:
>  = %a (List a ^{merge: ?xs (concat xs), empty: []});

### 合并

可合并类型:
> Score = Int ^{check: ?x (x >= 0 and x <= 100), merge2: ?x ?y (assume x == y; x)}; x : Score = 90; (x)  --->  90
> MergeList = %a (List a ^{merge: ?xs (concat xs)}); xs : MergeList Int = [1 2]; ys : MergeList Int = [3 4]; (x <+> y)  --->  [1 2 3 4]

默认合并逻辑:
> Int = Int ^{merge2: ?x ?y (assume x == y; x)};
> List = %a (List a ^{merge: ?xs (concat xs), empty: []});
> Set = %a (Set a ^{merge: ?xs (union xs), empty: []});
> Dict = %a (Dict a ^{merge: ?xs (unionDict xs), empty: {}});

重复声明等价于合并，但要求先标注类型（列表）:
> xs : MergeList Int; xs = [1 2]; xs = [3 4]; (xs)  --->  [1 2 3 4]

重复声明等价于合并，但要求先标注类型（对象）:
> cfg : Config; cfg.a = 1; cfg.b = 2; cfg   ===>   cfg : Config; cfg = {a = 1} <+> {b = 2}

### 错误处理

#### 函数实现错误

错误处理（assert关键字）:
> quicksort = ?xs (ys = quicksort-impl xs; assert ys == mergesort xs; ys); (quicksort [1,3,2])  --->  [1,2,3]

错误处理（error关键字）:
> quicksort = ?xs (ys = quicksort-impl xs; if (ys /= mergesort xs) then error "the sort result is wrong"; ys); (quicksort [1,3,2])  --->  [1,2,3]

错误处理, impossible 关键字, 用于逻辑上不可达的分支: 如果通过静态分析能确认它确实不可达, 则可以优化掉; 若无法确认, 则需要生成对应的运行时报错代码
> (if x >= 0 then 1 elif x < 0 then -1 else impossible)

#### 不合理的函数调用

错误处理（assume关键字）:
> div = ?a ?b (assume b != 0; a // b); (div 4 2)  --->  2

错误处理（blame关键字）:
> div = ?a ?b (if b == 0 then blame "The divisor cannot be zero" else a // b); (div 4 2)  --->  2

#### 推诿 (需要跟踪调用栈)

错误处理（deflect关键字）:
> quicksort = ?xs (ys = deflect (quicksort-impl xs); ys); (quicksort [1,3,2])  --->  [1,2,3]

### 运行时编译

opt 关键字用于AOT计算:
> x = opt (fib 20); x

opt 关键字用于JIT编译:
> f = ?a ?b fib a + b; g = opt (f 2)   ===>   f = ?a ?b a * a + b; g = ?b 4 + b;
> for (range 10) (?n (for (range 100) (?m (opt (f n) m))))   ===>  for (range 10) (?n (_g = (f n); for (range 100) (?m (_g m))))

### 委派

> div = ?(a: Int) ?(b: Int) (c: Int = a // b; c);
> div = ?(a: Float) ?(b: Float) (c: Float = a // b; c);
> ===>
> div : ?(a: Int) ?(b: Int) (c: Int; c) | ?(a: Float) ?(b: Float) (c: Float; c);
> div = ?a ?b (if (typefit a Int and typefit b Int) then (a // b) elif (typefit a Float and typefit b Float) then (a / b) else impossible)

支持快速解析的具体语法
------------------

1. 当遇到 `(` 时，开始一个新的表达式
2. 当遇到 字母时，开始一个新的标识符
    1. 判断是否关键字, 如果是则按对应关键字后续的可能性进行解析, 否则按后面跟的符号进行解析
    2. 如果后跟 = , 则开始一个新的let语句
    3. 如果后跟 : , 则开始一个新的类型声明
3. 当遇到 `{` 时，开始一个新的对象  (兼容JSON)
4. 当遇到 `[` 时，开始一个新的数组  (兼容JSON)
5. 当遇到 数字 时，开始一个新的数字字面量  (兼容JSON)
6. 当遇到 `"` 时，开始一个新的字符串字面量  (兼容JSON)
7. 当遇到 `?` 时，开始一个新的lambda函数  (兼容JSON)
