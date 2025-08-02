---
outline: deep
---

# Markdown Extension Examples

md完整语法 [full list of markdown extensions](https://vitepress.dev/zh/guide/markdown).

This page demonstrates some of the built-in markdown extensions provided by VitePress.

## 示例代码

内容1

### C# .NET

```csharp
int a = 1; //变量
string txt1 = "字符串"; //变量
var txt2 = "字符串"; //var自动类型推断
const int c = 2; //常量

//if语句
if (a > c) Console.WriteLine("a大于c");
else Console.WriteLine("a小于等于c");

//for循环
for (int i=0; i<3; i++)
{
  Console.WriteLine("i值:"+i);
}

//foreach循环
var list = new List<int> {1,2,3};
int[] list2 = {1, 2, 3};
int[] list3 = [1, 2, 3];
foreach (int i in list)
{
  Console.WriteLine("i值:" + i);
}
```

### Java

```java
int a = 1; //变量
String txt1 = "字符串"; //变量
var txt2 = "字符串"; //var自动类型推断 Java 10以上
final int c = 2; //常量

//if语句
if (a > c) System.out.println("a大于c");
else System.out.println("a小于等于c");

//for循环
for (int i=0; i<3; i++) 
{
  System.out.println("i值:"+i);
}

//foreach循环
int[] list = {1, 2, 3};
for (int i : list) {
  System.out.println("i值:" + i);
}
```

### 华为仓颉
```c
package cjdemo

main() {
  var a = 1 //可变的变量
  a = 0
  let b = 2 //不可变的变量
  const c = 3 //常量
  let txt1: String = "字符串" //变量
  let txt2 = "字符串" //let自动类型推断  
  var txt3 = "字符串" //var自动类型推断  

  //if语句 单行语句也必须用大括号
  if (a > c) { println("a大于c") } 
  else { println("a小于等于c") }

  //for循环 0..3表示0 1 2 而0..=3表示0 1 2 3
  for (i in 0..3) {
    println("${i}")
  }

  //foreach循环
  let list = [5, 0, 8]
  for (i in list) {
    println("${i}")
  }
  
}
```

### Python
```python
a = 1 #变量
txt1 = "字符串" #变量
txt2 = "字符串" #变量
c = 2 #常量

# if语句
if a > c:
   print("a大于c")
else:
   print("a小于等于c")

# for循环
for i in range(3):
   print("i值:", i)

# foreach循环
list = [1,2,3]
for i in list:
   print("i值:", i)
```

### Go
```go
package main

import (
 "fmt"
)

func main() {
 a := 1 //变量
 var a = 1 //变量
 txt1 := "字符串" //变量
 txt2 := "字符串" //变量
 const c int = 2 //常量

 //if语句
 if a > c {
  fmt.Println("a大于c")
 } else {
  fmt.Println("a小于等于c")
 }

 //for循环
 for i := 0; i < 3; i++ {
  fmt.Println("i值:", i)
 }

 //foreach循环
 list := []int{1, 2, 3}
 for _, i := range list {
  fmt.Println("i值:", i)
 }
}
```

### Rust
```rust
fn main() {
   let a = 1; //不可变 变量
   let txt1 = "字符串"; //不可变 变量
   let mut txt2 = "字符串"; //可变 变量
   const C: i32 = 2; //常量必须大写、必须指定类型

   //if语句
   if a > C {
   println!("a大于c");
   } else {
   println!("a小于等于c");
   }

   //for循环
   for i in 0..3 {
   println!("i值: {}", i);
   }

   //foreach循环
   let list = vec![1, 2, 3];
   for i in list {
   println!("i值: {}", i);
   }
}
```

### Kotlin
```kotlin
fun main() {
   val a = 1 //变量
   val txt1 = "字符串" //变量
   val txt2 = "字符串" //var自动类型
   const val c = 2 //常量

   //if语句
   if (a > c) {
   println("a大于c")
   } else {
   println("a小于等于c")
   }

   //for循环
   for (i in 0 until 3) {
   println("i值: $i")
   }

   //foreach循环
   val list = listOf(1, 2, 3)
   for (i in list) {
   println("i值: $i")
   }
}
```

### TypeScript
```typescript
let a: number = 1; //变量
let txt1: string = "字符串"; //变量
let txt2: string = "字符串"; //变量
const c: number = 2; //常量

// if语句
if (a > c) {
   console.log("a大于c");
} else {
   console.log("a小于等于c");
}

// for循环
for (let i = 0; i < 3; i++) {
   console.log("i值:", i);
}

// foreach循环
let list: number[] = [1, 2, 3];
for (let i of list) {
   console.log("i值:", i);
}
```


## Syntax Highlighting

VitePress provides Syntax Highlighting powered by [Shiki](https://github.com/shikijs/shiki), with additional features like line-highlighting:

**Input**

````md
```js{4}
export default {
  data () {
    return {
      msg: 'Highlighted!'
    }
  }
}
```
````

**Output**

```js{4}
export default {
  data () {
    return {
      msg: 'Highlighted!'
    }
  }
}
```

## Custom Containers

**Input**

```md
::: info
This is an info box.
:::

::: tip
This is a tip.
:::

::: warning
This is a warning.
:::

::: danger
This is a dangerous warning.
:::

::: details
This is a details block.
:::
```

**Output**

::: info
This is an info box.
:::

::: tip
This is a tip.
:::

::: warning
This is a warning.
:::

::: danger
This is a dangerous warning.
:::

::: details
This is a details block.
:::



## Results 2
内容2
### Theme Data 3.1
<pre>{{ theme }}</pre>

### 第3.2级
<pre>{{ page }}</pre>

#### 第4级
内容4
##### 第5.1级
内容5

## More

md完整语法 [full list of markdown extensions](https://vitepress.dev/zh/guide/markdown).
