---
outline: deep
---

# demo3

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

## More

Check out the documentation for the [full list of markdown extensions](https://vitepress.dev/zh/guide/markdown).
