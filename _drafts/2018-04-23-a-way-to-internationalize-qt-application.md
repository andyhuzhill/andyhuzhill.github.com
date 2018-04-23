---
layout: post
title: "一种国际化Qt应用程序的方法"
description: ""
category: Qt
tags: [Qt, Internationalization]
---

{% include JB/setup %}

Qt 是一个很方便的 C++ 应用开发框架(或许现在要加上 Qt Quick 开发框架?)，不仅仅是程序编写方便面，它提供了很多方便的类库，而且它也提供了很方便的国际化方案（也就是翻译成各国语言的方案）。

## 基本流程

### 编写代码阶段

我们先来说说在 Qt 中实现多国语言翻译需要使用的基本流程。首先我们需要在编写代码的时候就要使用 Qt 提供的翻译相关的函数来"包裹"住所有的需要翻译的字符串。

你说哪些才是需要翻译的字符串呢？就是任何会在用户界面上显示的字符串，如果不会显示自然就不需要翻译了。

如果你使用的是 C++ 代码，那么翻译用的函数就是`QObject::tr`函数。大多数时候，我们看到用到这个函数的时候可能都只是一个`tr`，因为都是在一个继承自 QObject 的类中，所以可以直接调用父类的成员函数了。如果是在自由函数中想使用的话就要把完整的函数名`QObject::tr`写全了。

如果你写的是 Qt Quick 的程序，这个函数就变成了 qsTr 或者是 qsTranslate 、qsTranslateNoOp ...


代码写完之后，需要在工程文件 (.pro 文件) 中添加需要翻译的源代码文件。

```
SOURCES += \
    main.cpp \
    mainwindow.cpp

TRANSLATIONS = \
    English.ts \
    Japanese.ts
```    

### 使用 lupdate 检索源代码生成翻译源文件

在工程目录下运行以下命令

{% highlight bash %}
lupdate project.pro
{% endhighlight %}

执行完后，目录下会出现 English.ts 和 Japanese.ts 两个文件，这两个文件就是由上文中工程文件中制定的文件名。

### 使用 linguist 进行翻译

然后就可以使用 Qt 提供的 lingust 工具打开上文生成的 .ts 文件，将对应的词条翻译成目标语言就好了。

### 使用 lrelease 生成最终程序使用的翻译文件

当 linguist 翻译完所有的词条之后，就需要生成最终给应用程序使用的二进制翻译文件了。
使用如下命令即可生成。

{% highlight bash %}
lrelease English.ts
{% endhighlight %}

执行完毕，目录下会多出一个 English.qm 文件，这个文件就是最终需要放在应用程序中分发给用户的。


## 问题

使用上面的流程就完成了一次基本的程序国际化的流程。一般来说程序的翻译工作和编码工作是由不同人来完成的，可能实际完成翻译的人不熟悉或者不愿去熟悉使用 linguist 的使用方法，他们提供给程序员的词条翻译可能就是一个 Excel 文件，这个 Excel 文件通常一列是待翻译的词条，另一列是相应的译文。

## 解决方法

于是我们可能需要一个将 Qt 的翻译文件与 Excel 文件互相转换的小工具，这样我们可以把 Qt 翻译文件转换成 Excel 文件交给翻译人员，然后将翻译人员翻译好的 Excel 文件再转换为 Qt 的翻译文件。

要做文件格式转换，那么我们首先得了解一下两种文件格式有什么特点。

### Qt TS 文件格式分析

Qt 的 TS 翻译文件实际是一个 XML 文件。我们先来看一个翻译文件的例子。

{% highlight xml %}
<!DOCTYPE TS>
<TS>
    <context>
        <name>QPushButton</name>
        <message>
            <source>Hello world!</source>
            <translation type="unfinished"></translation>
        </message>
    </context>
</TS>
{% endhighlight %}

这是一个还未翻译的文件，词条原文是"Hello world!"。在 ts 文件中，每个需要翻译的词条都是一个 message，message 包含需要翻译的原文和已翻译的译文。

翻译完成后需要去除 translation 的 `type="unfinished"` 属性。

### Excel 文件格式分析

Excel 的文件格式是 Microsoft &reg; 的私有格式，Excel 2007 之前的格式是二进制格式, Excel 2007 之后的是 OOXML 格式。

在这里我们不用太细的追究它的内部细节，因为它的内部细节非常复杂，不像 ts 文件就是一个简单的文本文件。现在有许多的可以用来读写 Excel 文件的库，我们只需调用这些库的读写函数就能完成我们所需的功能了。
在本文中，我将使用 C# 语言以及 NPOI 库来读写 Excel 文件。


### TS 文件转 Excel 文件


### Excel 转 TS 文件
