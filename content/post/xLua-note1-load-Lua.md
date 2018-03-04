---
title: "xLua学习笔记(1) 加载Lua代码"
date: 2018-03-01T10:15:17+08:00
draft: false
tags: ["Unity3D", "Lua", "xLua", "学习笔记"]
---
<!--more-->

# xLua的安装

首先从xLua的Github主页上下载资源包

https://github.com/Tencent/xLua

解压下载好的压缩包，将压缩包中Assets文件夹下的内容复制到Unity工程的Assets文件夹下即可

![Assets文件夹](/img/post/xLua-note1-load-Lua/img-0.png)

# 字符串加载

通过字符加载Lua代码的方式如下

```cs
using UnityEngine;
using XLua;

public class XLuaLoad : MonoBehaviour
{

    private LuaEnv luaenv;

	void Start ()
    {
        luaenv = new LuaEnv();
        object[] returns = luaenv.DoString
            (
                @"
                print('load lua by string.')
                return 1, 'hello'
                 "
            );
        Debug.Log("return value: " + returns[0] + " " + returns[1]);
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
}

```

将脚本挂载到任何一个游戏物体上，执行程序，可以得到


![](/img/post/xLua-note1-load-Lua/img-1.png)


这里使用到了LuaEnv对象的DoString方法，该方法的定义如下

```cs
object[] DoString(string chunk, string chunkName = "chuck", LuaTable env = null)
```

* 描述：

    执行一个代码块。

* 参数：

|参数|说明|
|:---|:---|
|chunk|Lua代码字符串|
|chunkName|发生error时的debug显示信息中使用，指明某某代码块的某行错误|
|env|这个代码块|

* 返回值：

代码块里return语句的返回值

>上面的代码中我们执行了
```cs
        object[] returns = luaenv.DoString
            (
                @"
                print('load lua by string.')
                return 1, 'hello'
                 "
            );
```
DoString返回将包含两个object， 一个是double类型的1， 一个是string类型的“hello”

# 加载Lua文件

新建一个lua文件，命名为 **my_script.lua** ，并向文件中写入下列测试代码

![](/img/post/xLua-note1-load-Lua/img-2.png)

接下来将 **my_script.lua** 文件放到 **Resources文件夹** 下，修改 **my_script.lua** 文件的 **后缀名** 为 **txt**

![](/img/post/xLua-note1-load-Lua/img-3.png)

然后通过以下方式加载 **my_script.lua** 文件中的Lua代码

```cs
using UnityEngine;
using XLua;

public class XLuaLoad : MonoBehaviour
{
    private LuaEnv luaenv;

	void Start ()
    {
        luaenv = new LuaEnv();
        luaenv.DoString
            (
                @"
                require 'my_script'
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
}
```

仍然是调用LuaEnv对象的DoString方法，不过这里通过使用Lua的全局函数 **require** 函数来加载其它的文件，在加载的时候会使用默认的Loader去加载文件

将脚本挂载到任何一个游戏物体上，执行程序，可以得到

![](/img/post/xLua-note1-load-Lua/img-4.png)

使用上述方法加载Lua代码时需要注意：

1. **Lua代码文件的文件名必须是XXX.lua.txt的格式，通过require引入的文件的时候只需要传入XXX**
2. **Lua代码文件必须放在Resources文件夹下，不能放在Resources下的某个文件夹里**

如果没有注意以上两点，运行的时候就会抛出异常，提示找不到文件

![](/img/post/xLua-note1-load-Lua/img-5.png)

如果想要把Lua代码文件放在任意文件夹下的话，就需要使用AddLoader方法添加自定义的Loader

```cs
void AddLoader(CustomLoader loader)
```

* 描述
增加一个自定义loader


CustomLoader的定义为

```cs
delegate byte[] CustomLoader(ref string filepath)
```

当一个文件被require时，这个loader会被回调其参数是require的参数，如果该loader找到文件，可以将其读进内存，返回一个byte数组。如果需要支持调试的话，而filepath要设置成IDE能找到的路径(相对或者绝对都可以)

需要注意的是，require的参数是什么，loader在回调的时候filepath就会被设置为什么

只需要对上面的代码进行略微修改即可

```cs
using UnityEngine;
using XLua;

public class XLuaLoad : MonoBehaviour
{
    private LuaEnv luaenv;

	void Start ()
    {
        luaenv = new LuaEnv();
        luaenv.AddLoader(LuaLoader);
        luaenv.DoString
            (
                @"
                require 'lua/my_script'
                 "
            );     
	}

    private byte[] LuaLoader(ref string filepath)
    {
        TextAsset script = Resources.Load("lua/" + filepath + ".lua") as TextAsset;
        return script.bytes;
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
}
```

在LuaLoader中可以使用任何方式来加载Lua代码，只要最后返回代码字符的byte数组即可

可以向LuaEnv中添加若干个Loader，在require的时候会依次调用每一个Loader，直到有一个Loader成功为止，如果全部失败则会抛出异常，提示找不到文件

建议的加载Lua脚本方式是：整个程序就一个DoString("require 'main'")，然后在main.lua加载其它脚本(类似lua脚本的命令行执行：lua main.lua)
