# 什么是soot
在参与研究JavaVMP保护技术的过程中，我们需要做一个Java语言向C语言转化的编译器。我们在寻找一种生成中间语言的工具来优化自己的编译器，soot就进入视野中了。

soot是一种java编译优化框架，可以利用它来对java字节码程序流和控制流进行分析。它的输入可以是一个class，一个dex甚至是一个apk，最终会返回给你想要的内容，可能是字节码，可能是基础的控制流图，可能是和C风格接近。

我这次使用了jimple和jasmin。

## 下载soot

官方链接：https://github.com/Sable/soot

相关官方文档很重要，里面写了一些详细用法。

如果没有科学上网，不太建议从github上直接下载源码。可以参考官方文档从eclipse或者IDEA导入soot项目。

## 使用soot

官方文档里写的很清楚，我这里强调几个坑。

①soot的path非常特殊，需要自己手动编译源码后，将一个名为sootclasses-trunk-jar-with-dependencies.jar的包的路径写在命令中

②如果是用jimple和jasmin等与java语义相关内容，还需要加上java的rt.jar路径
