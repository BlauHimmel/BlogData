---
title: "xLua学习笔记(2) C#调用Lua代码"
date: 2018-03-01
draft: false
tags: ["Unity3D", "Lua", "xLua", "学习笔记"]
---
<!--more-->

# 获取全局变量

只需要调用LuaEnv对象Global属性的Get方法即可

```cs
LuaTable Global;
```
* 描述：
代表lua全局环境的LuaTable

```cs
T Get<T>(string key);
```
* 描述：
获取在key下，类型为T的value，如果不存在或者类型不匹配，返回null；

例如有如下Lua代码

```lua
number = 1

string = "hello world"

boolean = true

```

在C#中尝试用上述方法输出Lua中的全局变量number，string和boolean

```cs
Debug.Log("number = " + luaenv.Global.Get<int>("number"));
Debug.Log("string = " + luaenv.Global.Get<string>("string"));
Debug.Log("boolean = " + luaenv.Global.Get<bool>("boolean"));
```

得到结果

![](/img/post/xLua-note2-CSharp-call-Lua/img-0.png)

如果需要获取Lua中 **Table** 的数据，则需要将Table映射为C#中相对应的数据结构，可选的方式有： **class** ， **interface** ， **Dictionary** ， **List** 和 **LuaTable**

假设有如下的Table

```lua
table =
{
    f1 = 1,
    f2 = 2
}
```

如果要使用 **class** 做映射，只需要在C#中定义一个相对应的类即可，注意变量名称要和Table中的名称相同，xLua会依次在table中寻找是否有与class中变量名相同的key值，如果有则将其value值复制到对应的变量上，如果在Table的Key值中找不到与定义的class的变量，则变量会被赋于对应类型的初值

```cs
class Table
{
    public int f1;
    public int f2;
}
```

获取变量的方法还是和之前一样

```cs
Table table = luaenv.Global.Get<Table>("table");
Debug.Log("table.f1 = " + table.f1);
Debug.Log("table.f2 = " + table.f2);
```

得到结果

![](/img/post/xLua-note2-CSharp-call-Lua/img-1.png)

同样地，还可以使用 **Dictionary** 和 **LuaTable** (速度比较慢)来映射Table，代码如下

```cs
Dictionary<string, int> dict = luaenv.Global.Get<Dictionary<string, int>>("table");
Debug.Log("dict[f1] = " + dict["f1"]);
Debug.Log("dict[f2] = " + dict["f2"]);

LuaTable luaTable = luaenv.Global.Get<LuaTable>("table");
Debug.Log("luaTable.Get<int>(\"f1\") = " + luaTable.Get<int>("f1"));
Debug.Log("luaTable.Get<int>(\"f2\") = " + luaTable.Get<int>("f2"));
```

得到结果

![](/img/post/xLua-note2-CSharp-call-Lua/img-2.png)

![](/img/post/xLua-note2-CSharp-call-Lua/img-3.png)


对于下面这种Table，可以使用 **List** 来做映射

```lua
table =
{
    1,
    2
}
```

```cs
List<int> list = luaenv.Global.Get<List<int>>("table");
Debug.Log("list[0] = " + list[0]);
Debug.Log("list[1] = " + list[1]);
```

得到结果

![](/img/post/xLua-note2-CSharp-call-Lua/img-4.png)

如果Table中存在函数，例如

```lua
table =
{
    f1 = 1,
    f2 = 2,
    add = function(self, num1, num2)
        return num1 + num2
    end
}
```

可以使用 **interface** 来映射，对于上述的Table，可以声明如下ITable接口

使用这种方法读取Table时需要 **生成代码** ，所以 **必须** 要给接口加上一个Attribute： **CSharpCallLua**

```cs
[CSharpCallLua]
interface ITable
{
    int f1 { get; set; }
    int f2 { get; set; }

    int add(int num1, int num2);
}
```

```cs
ITable iTable = luaenv.Global.Get<ITable>("table");
Debug.Log("table.f1 = " + iTable.f1);
Debug.Log("table.f2 = " + iTable.f2);
Debug.Log("table.add(1, 2) = " + iTable.add(1, 2));
```

得到结果

![](/img/post/xLua-note2-CSharp-call-Lua/img-5.png)

# 获取全局函数

一般来说，全局函数的映射方式有两种，一种是使用 **delegate** (要生成代码，性能好)，另一种是使用 **LuaFunction** (不用生成代码，性能较差)。

假设有下列的全局函数

```lua
function action0()
    print ("action0() called!")
end
```

因为使用 **delegate** 的方式映射全局函数需要生成代码，所以 **必须** 要添加一个Attribute： **CSharpCallLua**

```cs
[CSharpCallLua]
private delegate void Action0();
```

获取的方式大同小异

```cs
Action0 action0 = luaenv.Global.Get<Action0>("action0");
action0();
```

得到结果

![](/img/post/xLua-note2-CSharp-call-Lua/img-6.png)

当全局函数有一个或多个返回值的时候，按照在函数中的返回顺序， **从左到右** 依次对应到 **delegate** 的 **返回值** ， **ref** / **out** 参数

假设有下列的全局函数

```lua
function action2(param1, param2)
    print("param1:", param1, " param2:", param2)
    return 1, {f1 = 2}
end
```

那么下面六种 **delegate** 的映射方式是等效的，

```cs
[CSharpCallLua]
private delegate int Action2(int param1, int param2, out Table table);

[CSharpCallLua]
private delegate int Action2_(int param1, int param2, ref Table table);

[CSharpCallLua]
private delegate void Action2__(int param1, int param2, out int val ,out Table table);

[CSharpCallLua]
private delegate void Action2___(int param1, int param2, ref int val, ref Table table);

[CSharpCallLua]
private delegate void Action2____(int param1, int param2, ref int val, out Table table);

[CSharpCallLua]
private delegate void Action2_____(int param1, int param2, out int val, ref Table table);
```

**delegate** 的返回值也可以是 **delegate** ，假设有下列的全局函数

```lua
function action0()
    print ("action0() called!")
end

function get_action0()
    print("get_action0() called!")
    return action0
end
```

先定义Action0，然后将GetAction0的返回值设为Action0即可

```cs
[CSharpCallLua]
private delegate void Action0();

[CSharpCallLua]
private delegate Action0 GetAction0();
```

```lua
GetAction0 getAction0 = luaenv.Global.Get<GetAction0>("get_action0");
(getAction0())();
```

运行得到结果

![](/img/post/xLua-note2-CSharp-call-Lua/img-7.png)

使用 **LuaFunction** 来映射全局函数的方法与使用 **delegate** 时基本相同，映射后如果要在C#中使用函数，只需要调用 **LuaFunction** 对象的Call方法即可，Call方法有下面两种重载形式

```cs
object[] Call(params object[] args)
```
* 描述：
以可变参数调用Lua函数，并返回该调用的返回值。

```cs
object[] Call(object[] args, Type[] returnTypes)
```

* 描述：
调用Lua函数，并指明返回参数的类型，系统会自动按指定类型进行转换。

需要注意的是，当函数有返回值的时候，最好指明返回参数的类型，如果没有指明类型，在将object对象转换具体类型的时候可能会InvalidCastException异常

假设有下面的全局函数

```lua
function action2(param1, param2)
    print("param1:", param1, " param2:", param2)
    return 1, {f1 = 2}
end
```

下面的C#代码使用LuaFunction调用该全局函数

```cs
LuaFunction luaFunction = luaenv.Global.Get<LuaFunction>("action2");
object[] vals = luaFunction.Call(new object[]{1, 2}, new Type[] { typeof(int), typeof(Table) });
Debug.Log("vals[0] = " + (int)vals[0]);
Debug.Log("vals[1] = " + ((Table)vals[1]).f1);
```

得到结果

![](/img/post/xLua-note2-CSharp-call-Lua/img-8.png)

**LuaFunction** 访问Lua函数的过程涉及到了多次的装箱与拆箱操作，虽然不用生成代码，但是性能的消耗是比较大的

由于访问Lua全局数据，特别是table以及function，代价比较大，推荐使用单独的模块负责管理其加载，而不是每次用之前去加载

# 资源释放

下面这段这段C#代码首先将Lua函数test映射到了委托action上，使用完成后调用了LuaEnv的Dispose方法回收了LuaEnv对象

```cs
public class SpecialCase : MonoBehaviour
{
    [CSharpCallLua]
    public delegate void Action();

    private LuaEnv luaenv;
    private Action action;

	void Start ()
    {
        luaenv = new LuaEnv();
        luaenv.DoString
            (
                @"
                    function test()
                        print('test')
                    end
                "
            );
        action = luaenv.Global.Get<Action>("test");
        luaenv.Dispose();
	}
}
```

运行上面这段代码，发现抛出了如下的异常

![](/img/post/xLua-note2-CSharp-call-Lua/img-9.png)

查阅[FAQ](https://github.com/Tencent/xLua/blob/ad47200c213729368e68b590d0054610d225634c/Assets/XLua/Doc/faq.md#调用luaenvdispose时报try-to-dispose-a-luaenv-with-c-callback错是什么原因)

># 调用LuaEnv.Dispose时，报“try to dispose a LuaEnv with C# callback!”错是什么原因？

>这是由于C#还存在指向lua虚拟机里头某个函数的delegate，为了防止业务在虚拟机释放后调用这些无效(因为其引用的lua函数所在虚拟机都释放了)delegate导致的异常甚至崩溃，做了这个检查

>怎么解决？释放这些delegate即可，所谓释放，在C#中，就是没有引用

>你是在C#通过LuaTable.Get获取并保存到对象成员，赋值该成员为null

>你是在lua那把lua函数注册到一些事件事件回调，反注册这些回调

>如果你是通过xlua.hotfix(class, method, func)注入到C#，则通过xlua.hotfix(class, method, nil)删除

>要注意以上操作在Dispose之前完成

因此，只需要在LuaEnv释放之前，将Lua函数映射的委托变量action设置为null即可

```lua
action = luaenv.Global.Get<Action>("test");
action = null;
luaenv.Dispose();
```
