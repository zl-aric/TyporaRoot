# 🌳 **C# Expression Tree 结构总图（核心）**

```
Expression (抽象基类)
│
├── LambdaExpression                     // lambda 表达式
│   └── Expression<TDelegate>            // 强类型 lambda
│
├── ParameterExpression                  // 参数表达式
│
├── ConstantExpression                   // 常量
│
├── BinaryExpression                     // 二元运算
│   ├── Add / Subtract / Multiply        // 算术
│   ├── And / Or / ExclusiveOr           // 位运算
│   ├── AndAlso / OrElse                 // 逻辑短路
│   ├── Equal / NotEqual                 // 比较
│   ├── GreaterThan / LessThan           // 比较
│   └── Coalesce                         // ??
│
├── UnaryExpression                      // 一元运算
│   ├── Negate (-x)
│   ├── Not (!x)
│   └── Convert (类型转换)
│
├── MethodCallExpression                 // 方法调用
│
├── MemberExpression                     // 属性/字段访问
│   ├── PropertyExpression
│   └── FieldExpression
│
├── IndexExpression                      // arr[i] / 对象索引器
│
├── ConditionalExpression                // 条件表达式  x > 0 ? 1 : 0
│
├── BlockExpression                      // 代码块 { ... }
│
├── LoopExpression                       // 循环
│   ├── While
│   └── ForEach（可表达）
│
├── SwitchExpression                     // switch
│
├── TryExpression                        // try/catch/finally
│
├── NewExpression                        // new T()
│
├── MemberInitExpression                 // new T { ... }
│
├── NewArrayExpression                   // new [] { ... }
│   ├── NewArrayInit
│   └── NewArrayBounds
│
├── InvocationExpression                 // 调用 lambda：f(x)
│
├── TypeBinaryExpression                 // obj is T / obj as T
│
└── DefaultExpression                    // default(T)
```

------

# 📦 **第二层结构图：按用途分类**

下面是实际工作中最常见的“按逻辑用途”结构图，比官方更易懂：

------

## 📘 1. **基本元素（叶子节点）**

```
ParameterExpression     // x
ConstantExpression      // 5
DefaultExpression       // default(T)
```

------

## 🔧 2. **运算节点**

```
UnaryExpression         // !x, -x, (int)x
BinaryExpression        // x + y, x > y, x && y
```

------

## 📞 3. **成员/方法调用节点**

```
MemberExpression        // x.Name
MethodCallExpression    // x.ToString()
IndexExpression         // arr[0]
```

------

## 📦 4. **对象创建节点**

```
NewExpression           // new T()
MemberInitExpression    // new T { A = 1 }
NewArrayExpression      // new[] {1,2,3}
```

------

## 🔁 5. **流程控制节点**

```
ConditionalExpression   // if-else (三元)
BlockExpression         // { ... }
LoopExpression          // while(true)
SwitchExpression        // switch
TryExpression           // try/catch
```

------

## 🧠 6. **函数式表达式（核心）**

```
LambdaExpression        // x => x + 1
InvocationExpression    // f(5)
```

整个表达式树最终会组织成一个 LambdaExpression。

------

# 🎨 **超级直观结构图（完整版本，推荐收藏）**

```
Expression
├── 基础节点
│   ├── ConstantExpression
│   ├── ParameterExpression
│   └── DefaultExpression
│
├── 运算节点
│   ├── UnaryExpression
│   └── BinaryExpression
│
├── 成员与调用
│   ├── MemberExpression
│   ├── MethodCallExpression
│   ├── IndexExpression
│   └── InvocationExpression
│
├── 控制结构
│   ├── ConditionalExpression
│   ├── BlockExpression
│   ├── LoopExpression
│   ├── SwitchExpression
│   └── TryExpression
│
├── 面向对象
│   ├── NewExpression
│   ├── MemberInitExpression
│   └── NewArrayExpression
│
└── 最终表达式
    ├── LambdaExpression
    └── Expression<TDelegate>
```

------

# 🧩 **表达式树整体结构（景观图）**

```
                                   Expression
                                         │
                ┌────────────────────────┴────────────────────────┐
                │                                                 │
         值表达式（Value）                                   行为表达式（Statement-like）
                │                                                 │
       ┌────────┴─────────┐                            ┌─────────┴───────────┐
       │                  │                            │                     │
   简单值节点         计算节点                   控制结构节点            对象操作节点
       │                  │                            │                     │
 Constant / Parameter   Unary / Binary    If / Loop / Switch / Try     New / MemberInit / Array
```

------

# 