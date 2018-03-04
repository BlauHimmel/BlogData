---
title: "xLua学习笔记(3) Lua调用C#代码"
date: 2018-03-01T10:35:17+08:00
draft: false
tags: ["Unity3D", "Lua", "xLua", "学习笔记"]
---
<!--more-->

将下列代码挂载到任何一个GameObject上，这样就能在Unity中加载并执行 **Resources/Lua** 文件夹下的 **csharp_call.lua.txt** 文件中Lua代码了

```cs
public class CSharpRun : MonoBehaviour
{
    private LuaEnv luaenv;

	void Start ()
    {
        luaenv = new LuaEnv();
        luaenv.AddLoader(LuaLoader);
        luaenv.DoString
            (
                @"
                require 'csharp_call'
                "
            );
	}

	void Update ()
    {
	    if (luaenv != null)
        {
            luaenv.Tick();
        }
	}

    void Destroy()
    {
        if (luaenv != null)
        {
            luaenv.Dispose();
        }
    }

    private byte[] LuaLoader(ref string filename)
    {
        TextAsset text = Resources.Load("Lua/" + filename + ".lua") as TextAsset;
        return text.bytes;
    }
}
```

# 访问C#方法、属性

使用下列Lua代码来创建两个GameObject对象，访问对象的name属性并调用其SetActive方法

```lua
--创建C#对象
local go1 = CS.UnityEngine.GameObject("Cube")
local go2 = CS.UnityEngine.GameObject("Sphere")

--访问对象属性
print("GameObject1:", go1.name, "GameObject2:", go2.name)

--修改对象属性
go1.name = "Cube--"
go2.name = "Sphere--"
print("GameObject1:", go1.name, "GameObject2:", go2.name)

--调用对象方法(调用对象方法时注意使用":"语法糖)
go1:SetActive(false)  --go1.SetActive(go1, false) 也可以这样调用
go2:SetActive(false)  --go2.SetActive(go2, false) 也可以这样调用
```

运行得到结果，GameObject的名字被成功地修改

![](/img/post/xLua-note3-Lua-call-CSharp/img-0.png)

游戏物体Cube和Sphere被创建到了Hierarchy下，同时被设置Enable状态

![](/img/post/xLua-note3-Lua-call-CSharp/img-1.png)

# 访问C#静态方法、静态属性

下面的Lua代码：
1. 获取Time类的引用
2. 读取Time类的静态属性deltaTime
3. 修改了Time类的静态属性timeScale
4. 获取GameObject类的引用
5. 调用GameObject类的静态方法Find，找到场景中的主相机Main Camera

```lua
--读取静态属性
local Time = CS.UnityEngine.Time
print("Time.deltaTime:", Time.deltaTime)

--修改静态属性
print("Time.timeScale Before:", Time.timeScale)
Time.timeScale = 0.5
print("Time.timeScale After:", Time.timeScale)

--调用静态方法(调用静态方法时可以直接使用".")
local GameObject = CS.UnityEngine.GameObject
local mainCamera = GameObject.Find("Main Camera")
print("mainCamera:", mainCamera.name)
```

运行得到结果

![](/img/post/xLua-note3-Lua-call-CSharp/img-2.png)

# 访问自定义的C#类

如果要通过Lua访问自己定义的类的话，需要给被Lua代码访问的类加上一个Attribute: **LuaCallCSharp** ，用来生成Lua的适配代码

下面对 **LuaCallCSharp** 的解释参考xLua的Github主页的[FAQ](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Doc/faq.md#luacallsharp以及csharpcalllua)

># XLua.LuaCallCSharp

>一个C#类型加了这个配置，xLua会生成这个类型的适配代码(包括构造该类型实例，访问其成员属性、方法，静态属性、方法)，否则将会尝试用性能较低的反射方式来访问。

>一个类型的扩展方法(Extension Methods)加了这配置，也会生成适配代码并追加到被扩展类型的成员方法上。

>xLua只会生成加了该配置的类型，不会自动生成其父类的适配代码，当访问子类对象的父类方法，如果该父类加了LuaCallCSharp配置，则执行父类的适配代码，否则会尝试用反射来访问。

>反射访问除了性能不佳之外，在il2cpp下还有可能因为代码剪裁而导致无法访问，后者可以通过下面介绍的ReflectionUse标签来避免。

># XLua.ReflectionUse

>一个C#类型类型加了这个配置，xLua会生成link.xml阻止il2cpp的代码剪裁。

>对于扩展方法，必须加上LuaCallCSharp或者ReflectionUse才可以被访问到。

>建议所有要在Lua访问的类型，要么加LuaCallCSharp，要么加上ReflectionUse，这才能够保证在各平台都能正常运行。


下面定义 **BaseClass** 和 **DeriveClass** ，其中 **BaseClass** 是 **DeriveClass** 的基类

```cs
[LuaCallCSharp]
class BaseClass
{
    public string BaseField = "BaseField";

    public string _BaseProperty = "BaseProperty";
    public string BaseProperty
    {
        set { _BaseProperty = value; }
        get { return _BaseProperty; }
    }

    public static string BaseStatic = "BaseStatic";

    public static string _BaseStaticProperty = "BaseStaticProperty";
    public static string BaseStaticProperty
    {
        set { _BaseStaticProperty = value; }
        get { return _BaseStaticProperty; }
    }

    public void BasePrint()
    {
        Debug.Log("BasePrint() Called.");
    }

    public static void BaseStaticPrint()
    {
        Debug.Log("BaseStaticPrint() Called.");
    }
}

[LuaCallCSharp]
class DeriveClass : BaseClass
{
    public string DeriveField = "DeriveField";

    public string _DeriveProperty = "BaseDeriveProperty";
    public string DeriveProperty
    {
        set { _DeriveProperty = value; }
        get { return _DeriveProperty; }
    }

    public static string DeriveStatic = "DeriveStatic";

    public static string _DeriveStaticProperty = "DeriveStaticProperty";
    public static string DeriveStaticProperty
    {
        set { _DeriveStaticProperty = value; }
        get { return _DeriveStaticProperty; }
    }

    public void DerivePrint()
    {
        Debug.Log("DerivePrint() Called.");
    }

    public static void DeriveStaticPrint()
    {
        Debug.Log("DeriveStaticPrint() Called.");
    }
}
```

访问的方式和之前访问Unity中类的方式相同，另外xLua中除了能够 **访问** 和 **修改** 类自身的成员以外，还支持在派生类中 **访问** 和 **修改** 基类的成员，访问的规则和C#中相同

另外，访问权限不足时，Lua中相应的变量会被赋值为nil

```lua
local BaseClass = CS.BaseClass
local baseClass = CS.BaseClass()

local DeriveClass = CS.DeriveClass
local deriveClass = CS.DeriveClass()

print('----------Base----------')
print(baseClass.BaseField)
print(baseClass.BaseProperty)
baseClass:BasePrint()

print(BaseClass.BaseStatic)
print(BaseClass.BaseStaticProperty)
BaseClass.BaseStaticPrint()

print('---------Derive---------')
print(deriveClass.DeriveField)
print(deriveClass.DeriveProperty)
deriveClass:DerivePrint()

print(DeriveClass.DeriveStatic)
print(DeriveClass.DeriveStaticProperty)
DeriveClass.DeriveStaticPrint()

print('------Derive->Base------')
print(deriveClass.BaseField)
print(deriveClass.BaseProperty)
deriveClass:BasePrint()

print(DeriveClass.BaseStatic)
print(DeriveClass.BaseStaticProperty)
DeriveClass.BaseStaticPrint()
```

# 访问C#复杂函数

* 对于参数，Lua会 **从左到右** 取C#函数中的 **普通参数** 或者 **ref参数** 依次作为自己的参数

* 对于返回值，Lua会 **从左到右** 取C#函数中的 **返回值** 、 **ref参数** 或者 **out参数** 依次作为自己的返回值

例如，对于下面的函数ComplexFunction

```cs
[LuaCallCSharp]
class ComplexClass
{
    public string ComplexFunction(int arg0, ref int arg1, string arg2, out string arg3, Parameter param)
    {
        Debug.Log("=========C#=========");
        Debug.Log("arg0:" + arg0);
        Debug.Log("arg1:" + arg1);
        Debug.Log("arg2:" + arg2);
        Debug.Log("arg3:");
        Debug.Log("Parameter:" + param);

        arg1++;
        arg3 = "3(string)";

        return "ComplexFunction return.";
    }
}

[LuaCallCSharp]
class Parameter
{
    public int param1;
    public string param2;

    public override string ToString()
    {
        return string.Format("param1:{0} param2:{1}", param1, param2);
    }
}
```

按照上面的原则，首先 **从左到右** 检索函数ComplexFunction的普通参数或ref参数

结果为：arg0,arg1,arg2,param

所以在Lua中调用时需要传入以上四个参数

接下来 **从左到右** 检索函数的返回值、ref参数或者out参数

结果为：ComplexFunction函数的返回值,arg1,arg3

所以在Lua中将返回上述4个值

```lua
local complexClass = CS.ComplexClass()
local ret, arg1, arg3 = complexClass:ComplexFunction(
    0,
    1,
    "2(string)",
    {param1 = 3, param2 = "4(string)"}
    )

print("=========Lua========")
print("ret:", ret)
print("arg1:", arg1)
print("arg3:", arg3)
```

运行得到结果

![](/img/post/xLua-note3-Lua-call-CSharp/img-3.png)

# 操作符重载和函数重载

C#中定义的操作符重载和函数重载在Lua中基本上能够使用，不过需要注意的是由于Lua中表示数值的类型只有一种(number)，所以C#中对于数值类型之间的重载是不能够正确的识别的，通常只会调用类型符合的重载函数列表中先定义的函数

假设有下面两个C#类，在Vector类中重载了操作符"+"，在Overload类中对函数Add进行了重载，类型分别是int，float和string

```cs
[LuaCallCSharp]
class Vector
{
    public int x;
    public int y;

    public Vector(int x, int y)
    {
        this.x = x;
        this.y = y;
    }

    public static Vector operator +(Vector vec1, Vector vec2)
    {      
        return new Vector(vec1.x + vec2.x, vec1.y + vec2.y);
    }
}

[LuaCallCSharp]
class Overload
{
    public int Add(int num1, int num2)
    {
        Debug.Log("Add-int");
        return num1 + num2;
    }

    public float Add(float num1, float num2)
    {
        Debug.Log("Add-float");
        return num1 + num2;
    }

    public string Add(string num1, string num2)
    {
        Debug.Log("Add-string");
        return num1 + num2;
    }
}
```

接下来使用Lua来访问进行验证，首先定义了两个Vector并相加，接着尝试向Overload的Add函数中传入整数，浮点数和字符串

```lua
local vec1 = CS.Vector(1,1)
local vec2 = CS.Vector(2,3)
local vec3 = vec1 + vec2
print("x:", vec3.x, "y:", vec3.y)

local overload = CS.Overload()
overload:Add(1, 1)
overload:Add(2.0, 2.0)
overload:Add("3", "3")
```

通过运行可以看到结果如下

![](/img/post/xLua-note3-Lua-call-CSharp/img-4.png)

由于在Lua中只有一种数值类型(number)，所以参数为int和float类型的Add函数都满足要求，这个时候会调用先定义的重载函数，也就是重载为int类型的Add

当先定义参数float类型后定义int类型的Add函数时，Lua代码调用就是参数类型为float的Add函数了

```cs
[LuaCallCSharp]
class Overload
{
    public float Add(float num1, float num2)
    {
        Debug.Log("Add-float");
        return num1 + num2;
    }

    public int Add(int num1, int num2)
    {
        Debug.Log("Add-int");
        return num1 + num2;
    }

    public string Add(string num1, string num2)
    {
        Debug.Log("Add-string");
        return num1 + num2;
    }
}
```

更换顺序后结果发生了变化

![](/img/post/xLua-note3-Lua-call-CSharp/img-5.png)

# 可变参数与默认参数

定义SpecialParam类，包含一个有默认参数的函数和一个有可变参数的函数

```cs
[LuaCallCSharp]
class SpecialParam
{
    public void DefaultParam(int arg0, string arg1 = "1", int arg2 = 2)
    {
        Debug.Log("arg0:" + arg0);
        Debug.Log("arg1:" + arg1);
        Debug.Log("arg2:" + arg2);
    }

    public void VariableParam(int arg0, params string[] args)
    {
        Debug.Log("arg0:" + arg0);
        Debug.Log("args:");
        foreach (string arg in args)
        {
            Debug.Log(arg + " ");
        }
    }
}
```

在Lua中调用它们的时候，参数的规则与C#中相同

```lua
local specialParam = CS.SpecialParam()
specialParam:DefaultParam(1, "3")
print("================================")
specialParam:VariableParam(0, "1", "2", "3", "4")
```

输出结果

![](/img/post/xLua-note3-Lua-call-CSharp/img-6.png)

# 访问C#枚举

定义枚举Language和EnumParam类，EnumParam类中的PrintEnum函数会根据传入枚举的类型输出不同的日志

```cs
[LuaCallCSharp]
public enum Language
{
    C_PLUS_PLUS,
    C_SHARP
}

[LuaCallCSharp]
class EnumParam
{
    public void PrintEnum(Language language)
    {
        switch (language)
        {
            case Language.C_PLUS_PLUS:
                Debug.Log("C++");
                break;
            case Language.C_SHARP:
                Debug.Log("C#");
                break;
        }
    }
}
```

在Lua中有以下五种方法可以访问到枚举变量：
1. 当作普通的静态属性访问
2. 使用__CastFrom函数，从枚举值对应的数值做类型转换
3. 使用__CastFrom函数，从枚举值对应的字符串做类型转换
4. 直接传入枚举值对应的数值
5. 直接传入枚举值对应的字符串

官方文档上只提到了前三种方法，并且方法2和方法3需要生成代码才能使用，不过经过实验发现，不生成代码时上述五种方法都能够使用

```lua
local enumParam = CS.EnumParam()
local Language = CS.Language

enumParam:PrintEnum(Language.C_PLUS_PLUS)
enumParam:PrintEnum(Language.__CastFrom(0))
enumParam:PrintEnum(Language.__CastFrom("C_PLUS_PLUS"))
enumParam:PrintEnum(0)
enumParam:PrintEnum("C_PLUS_PLUS")

enumParam:PrintEnum(Language.C_SHARP)
enumParam:PrintEnum(Language.__CastFrom(1))
enumParam:PrintEnum(Language.__CastFrom("C_SHARP"))
enumParam:PrintEnum(1)
enumParam:PrintEnum("C_SHARP")
```

输出结果

![](/img/post/xLua-note3-Lua-call-CSharp/img-7.png)

# 访问C#委托

下面的DelegateClass类定义了一个接受string类型参数无返回值的委托类型，3个委托变量action，actionString1和actionString2

```cs
[LuaCallCSharp]
class DelegateClass
{
    public delegate void ActionString(string arg);

    public ActionString action = (arg) =>
        {
            Debug.Log("action:" + arg);
        };

    public ActionString actionString1 = (arg) =>
        {
            Debug.Log("actionString1:" + arg);
        };

    public ActionString actionString2 = (arg) =>
        {
            Debug.Log("actionString2:" + arg);
        };
}
```

在使用Lua代码访问C#委托时需要注意，访问委托类型的方式与访问静态变量的方式相同，访问(静态/非静态)委托的变量的方式与访问(静态/非静态)成员变量的方式相同

由于在Lua中没有"+="和"-="操作符，在增加委托链的时候只能使用"+"和"-"操作符

```lua
local delegateClass = CS.DelegateClass()
--使用DelegateClass类的对象访问委托变量action
local action1 = delegateClass.action
action1("hi-1")
action1 = action1 + delegateClass.actionString1 + delegateClass.actionString2
action1("hi-2")
action1 = action1 - delegateClass.actionString2
action1("hi-3")

--使用DelegateClass类访问委托类型ActionString，定义一个ActionString类型的委托变量action2
--此时action2的值为nil
local action2 = CS.DelegateClass.ActionString
action2 = delegateClass.actionString1
action2("hi-4")
action2 = action2 + delegateClass.actionString2
action2("hi-5")
action2 = action2 - delegateClass.actionString2
action2("hi-6")
```

输出结果

![](/img/post/xLua-note3-Lua-call-CSharp/img-8.png)

在增减委托链的时候除了可以使用C#委托变量外，还可以使用Lua函数

```lua
function lua_action(arg)
    print("lua_action:", arg)
end

local action = CS.DelegateClass.ActionString
action = lua_action
action("hi")
```

访问C#事件

下面的EventClass类定义了一个无参数无返回值的委托类型EventAction和基于委托类型EventAction的事件Events，同时提供了两个定义好的委托类型action1和action2，和一个触发事件的函数TriggerEvent

```cs
[LuaCallCSharp]
class EventClass
{
    [CSharpCallLua]
    public delegate void EventAction();

    public event EventAction Events;

    public EventAction action1 = () =>
        {
            Debug.Log("action1");
        };

    public EventAction action2 = () =>
        {
            Debug.Log("action2");
        };

    public void TriggerEvent()
    {
        Events();
    }
}
```

在访问C#事件的时候需要 **生成代码** ，所以必须要为事件的委托类型加上一个Attribute： **CSharpCallLua** ，关于为什么这里需要加 **CSharpCallLua** 而不是LuaCallCSharp，在xLua的github主页的[FAQ](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Doc/faq.md#luacallsharp以及csharpcalllua)上作者是这么解释的：

># LuaCallCSharp以及CSharpCallLua两种生成各在什么场景下用？

>看调用者和被调用者，比如要在lua调用C#的GameObject.Find函数，或者调用gameobject的实例方法，属性等，GameObject类要加LuaCallSharp，而想把一个lua函数挂到UI回调，这是调用者是C#，被调用的是一个lua函数，所以回调声明的delegate要加CSharpCallLua。

>有时会比较迷惑人，比如List.Find(Predicate match)的调用，List当然是加LuaCallSharp，而Predicate却要加CSharpCallLua，因为match的调用者在C#，被调用的是一个lua函数。

>更无脑一点的方式是看到“This delegate/interface must add to CSharpCallLua : XXX”，就把XXX加到CSharpCallLua即可。

在添加事件的时候，既可以使用C#中的委托变量，也可以使用Lua中的函数

同时在添加和移除事件的时候应该使用以下的方式

```lua
object:event("+", delegate)
```
 ```lua
object:event("-", delegate)
```

```lua
function lua_action()
    print("lua_action:")
end

local eventClass = CS.EventClass()
eventClass:Events("+", lua_action)
eventClass:Events("+", eventClass.action1)
eventClass:Events("+", eventClass.action2)
eventClass:TriggerEvent()
```

运行结果

![](/img/post/xLua-note3-Lua-call-CSharp/img-9.png)

注意在Lua中不能通过以下方式来触发事件

```lua
eventClass:Events()
```

因为Events此时只是一个记录事件委托链的Table，并不是一个函数

# 类型信息与泛型方法

一个很简单的需求就是我们想要给新创建的GameObject添加某一个组件

在C#中一般的做法是使用AddComponent函数

```cs
gameObject.AddComponent<Rigidbody>();
gameObject.AddComponent("Rigidbody");
gameObject.AddComponent(typeof(Rigidbody));
```
AddComponent函数有三种重载形式，可以通过泛型、类名字符串和类的类型信息Type对象三种方式来为一个GameObject对象添加一个组件

而xLua不支持泛型，如果想要调用只能通过定义扩展方法，然后在Lua中通过调用扩展方法的方式来进行间接地调用

```cs
[LuaCallCSharp]
static class ExtendedMethod
{
    public static Rigidbody AddComponentRigidbody(this GameObject gameobject)
    {
        return gameobject.AddComponent<Rigidbody>();
    }

    [LuaCallCSharp]
    public static List<Type> luaCallCSharpList = new List<Type>()
    {
        typeof(GameObject),
    };
}
```

由于GameObject是Unity的API，不能修改其源代码，所以不能通过在其上添加 **LuaCallCSharp** ，所以这里使用了[另外一种方法](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Doc/configure.md)

># 静态列表

>有时我们无法直接给一个类型打标签，比如系统api，没源码的库，或者实例化的泛化类型，这时你可以在一个静态类里声明一个静态字段，该字段的类型除BlackList和AdditionalProperties之外只要实现了IEnumerable<Type>就可以了(这两个例外后面具体会说)，然后为这字段加上标签：

```cs
[LuaCallCSharp]
public static List<Type> mymodule_lua_call_cs_list = new List<Type>()
{
    typeof(GameObject),
    typeof(Dictionary<string, int>),
};
```

这样就能通过Lua代码间接调用AddComponent的泛型重载形式来添加Rigidbody组件

```lua
local go = CS.UnityEngine.GameObject("SuperCube")
go:AddComponentRigidbody()
```

AddComponent的字符串重载形式则可以在Lua代码中直接调用

```lua
local go = CS.UnityEngine.GameObject("SuperCube")
go:AddComponent("Animator")
```

对于传入Type的第三种重载形式，xLua在Lua API中为我们提供了一个和C#中typeof函数一样的函数，在Lua代码中也可以通过typeof得到类的类型信息

```lua
local go = CS.UnityEngine.GameObject("SuperCube")
go:AddComponent(typeof(CS.UnityEngine.Rigidbody))
```

# 类型转换

很多第三方库对外只暴露接口，而但我们通过Lua来调用的时候，这些没有对外暴露类只能够通过反射的方式来进行访问，如果这个接口被频繁地调用，势必会影响性能

为了提高运行效率，可以使用之前提到的静态列表的方式，将第三方库对外暴露的接口加入到代码生成列表中，生成Lua适配代码，这样然后在Lua中把具体的实现类转换为接口，然后通过接口来调用C#代码

下面举一个列子，假设某个第三方库是这样的

```cs
[LuaCallCSharp]
interface IFuckable
{
    void Fuck();
}

[LuaCallCSharp]
class Lisa : IFuckable
{
    public int id = 100;

    public void Fuck()
    {
        Debug.Log("Lisa[" + id + "] Can Fuck!");
    }
}

/*IFuckable的实现类很多*/

[LuaCallCSharp]
class WhoreHouse
{
    public IFuckable GetWhore()
    {
        return new Lisa();
    }
}
```

其中IFuckable和WhoreHouse是该库对外暴露的接口，外部通过调用WhoreHouse的GetWhore方法来得到不同的实现了IFuckable接口的对象

在Lua代码中，我们通过GetWhore拿到了一个woman对象，但是由于不知道woman到底是哪一个具体的实现类，所以直接调用的时候xLua会通过反射的方式来访问该实现类

为了通过IFuckable接口来进行调用，需要在Lua中将得到的woman对象转换为IFuckable接口类型

xLua为我们提供了一个类型转换函数 **cast** ，该函数有两个参数：
1. 需要进行类型转换的对象
2. 转换类型的Type对象(使用之前提到的**typeof**函数得到Type对象)

```lua
local whorehouse = CS.WhoreHouse()
local woman = whorehouse:GetWhore()
woman:Fuck()
cast(woman, typeof(CS.IFuckable))
woman:Fuck()
```

从输出的结果中可以看出，转换后实现类独有的字段仍然id能够正确的输出

![](/img/post/xLua-note3-Lua-call-CSharp/img-10.png)
