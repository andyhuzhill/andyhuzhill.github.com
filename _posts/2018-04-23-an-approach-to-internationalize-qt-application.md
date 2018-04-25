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

Qt 的 TS 翻译文件其实就是一个 XML 文件。我们先来看一个翻译文件的例子。

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
{% highlight csharp %}
private void convertQtFile2ExcelFile(string qtFileName, string excelFileName)
{
    XmlDocument xmlDoc = new XmlDocument();
    XmlReaderSettings settings = new XmlReaderSettings();
    settings.IgnoreComments = true;
    settings.DtdProcessing = DtdProcessing.Ignore;

    IWorkbook workbook = new XSSFWorkbook();
    ISheet sheet = workbook.CreateSheet("worksheet");

    int rowCnt = 0;
    IRow headRow = sheet.CreateRow(rowCnt);
    rowCnt++;

    ICell chCell = headRow.CreateCell(0);
    chCell.SetCellValue("源语言");
    ICell enCell = headRow.CreateCell(1);
    enCell.SetCellValue("目标语言");

    XmlReader reader = XmlReader.Create(qtFileName, settings);
    xmlDoc.Load(reader);

    XmlNode rootNode = xmlDoc.SelectSingleNode("TS");
    XmlNodeList xnl = rootNode.ChildNodes;

    foreach (XmlNode node in xnl)
    {
        XmlNodeList nodeList = node.ChildNodes;
        string src = "";
        string dst = "";
        foreach (XmlNode n in nodeList)
        {
            foreach (XmlNode cn in n.ChildNodes)
            {
                if (cn.Name == "source")
                {
                    src = cn.InnerText;
                }
                if (cn.Name == "translation")
                {
                    dst = cn.InnerText;
                    IRow row = sheet.CreateRow(rowCnt);
                    rowCnt++;
                    ICell firstCell = row.CreateCell(0);
                    firstCell.SetCellValue(src);
                    ICell secondCell = row.CreateCell(1);
                    secondCell.SetCellValue(dst);
                    //Console.WriteLine ("Src = " + src + " dst = " + dst);
                }
            }
        }
    }

    reader.Close();

    using (FileStream fs = File.Create(excelFileName))
    {
        workbook.Write(fs);
    }
}
{% endhighlight %}

### Excel 转 TS 文件

{% highlight csharp %}
private bool convertExcelFile2QtFile(string excelFileName, string qtFileName)
{
    try
    {
        using (var fs = File.OpenRead(excelFileName))
        {
            var workBook = new XSSFWorkbook(fs);
            var sheet = workBook.GetSheetAt(0);
            var translateMap = new Dictionary<string, string>();

            for (int i = 1; i < sheet.LastRowNum; i++)
            {
                IRow row = sheet.GetRow(i);

                if (row == null)
                {
                    continue;
                }

                var srcCell = row.GetCell(0);
                var dstCell = row.GetCell(1);

                if (srcCell == null)
                {
                    continue;
                }
                if (dstCell == null)
                {
                    continue;
                }

                string src = srcCell.ToString();
                string translated = dstCell.ToString();

                if (translateMap.ContainsKey(src) == false)
                {
                    translateMap.Add(src, translated);
                }
            }

            var document = new XmlDocument();
            document.Load(qtFileName);

            XmlNodeList xnl = document.SelectNodes("/TS/context");

            if (xnl != null)
            {
                foreach (XmlNode ctxNode in xnl)
                {
                    var msgList = ctxNode.SelectNodes("message");
                    foreach (XmlNode msg in msgList)
                    {
                        string src = "";

                        foreach (XmlNode cn in msg.ChildNodes)
                        {
                            if (cn.Name == "source")
                            {
                                src = cn.InnerText;
                            }
                            else if (cn.Name == "translation")
                            {
                                if (translateMap.ContainsKey(src))
                                {
                                    cn.InnerText = translateMap[src];
                                    var attrs = cn.Attributes;
                                    attrs.Remove(attrs["type"]);
                                }
                            }
                        }
                    }
                }
            }

            document.Save(qtFileName);
        }
        return true;
    }
    catch (Exception e)
    {
        MessageBox.Show("Exception:" + e.Message);
        return false;
    }
}

{% endhighlight %}
         
## 自动翻译词条

其实了解了 TS 文件的结构之后，我们还能做一件事，那就是直接调用网上的翻译接口，自动将所有的词条翻译成对应的语言，下面就是一个简单的例子。

{% highlight python %}
import xml.etree.ElementTree as ET
import os

import httplib
import md5
import urllib
import random
import json
import threading

import requests
from PyQt4.QtCore import *
from PyQt4.QtGui import *

import sys
reload(sys)
sys.setdefaultencoding('utf-8')


## 下面填写你自己的百度翻译 API 的 appid 和 secretKey
appid = ''
secretKey = ''

myurl = '/api/trans/vip/translate'

def translate(string, targetLanguage):
    if string is None:
        return ""
    salt = random.randint(32768, 65536)
    sign = appid + string + str(salt) + secretKey
    m1 = md5.new()
    m1.update(sign)
    sign = m1.hexdigest()
    encoded_str = urllib.quote(str(string))
    localurl = myurl + '?appid=' + appid + '&q=' + \
        encoded_str + '&from=auto&to=' + \
        targetLanguage + '&salt=' + str(salt) + '&sign=' + sign

    try:
        r = requests.get('http://api.fanyi.baidu.com/' + localurl)

        if r.status_code == 200:
            response = r.text
            objResponse = json.loads(response)
            return objResponse["trans_result"][0]["dst"]
    except Exception, e:
        print e
   
    return string


class MainWidget(QWidget):
    def __init__(self, parent=None):
        super(QWidget, self).__init__(parent)
        self.setWindowTitle(u"UI Auto Translator")
        self.labelTargetLanguage = QLabel(u"目标语言:")
        self.comboTargetLanguage = QComboBox()
        self.comboTargetLanguage.addItem(u"中文", u"zh")
        self.comboTargetLanguage.addItem(u"英语", u"en")
        self.comboTargetLanguage.addItem(u"粤语", u"yue")
        self.comboTargetLanguage.addItem(u"文言文", u"wyw")
        self.comboTargetLanguage.addItem(u"日语", u"jp")
        self.comboTargetLanguage.addItem(u"韩语", u"kor")
        self.comboTargetLanguage.addItem(u"法语", u"fra")
        self.comboTargetLanguage.addItem(u"西班牙语", u"spa")
        self.comboTargetLanguage.addItem(u"泰语", u"th")
        self.comboTargetLanguage.addItem(u"阿拉伯语", u"ara")
        self.comboTargetLanguage.addItem(u"俄语", u"ru")
        self.comboTargetLanguage.addItem(u"葡萄牙语", u"pt")
        self.comboTargetLanguage.addItem(u"德语", u"de")
        self.comboTargetLanguage.addItem(u"意大利语", u"it")
        self.comboTargetLanguage.addItem(u"希腊语", u"el")
        self.comboTargetLanguage.addItem(u"荷兰语", u"nl")
        self.comboTargetLanguage.addItem(u"波兰语", u"pl")
        self.comboTargetLanguage.addItem(u"保加利亚语", u"bul")
        self.comboTargetLanguage.addItem(u"爱沙尼亚语", u"est")
        self.comboTargetLanguage.addItem(u"丹麦语", u"dan")
        self.comboTargetLanguage.addItem(u"芬兰语", u"fin")
        self.comboTargetLanguage.addItem(u"捷克语", u"cs")
        self.comboTargetLanguage.addItem(u"罗马尼亚语", u"rom")
        self.comboTargetLanguage.addItem(u"斯洛文尼亚语", u"slo")
        self.comboTargetLanguage.addItem(u"瑞典语", u"swe")
        self.comboTargetLanguage.addItem(u"匈牙利语", u"hu")
        self.comboTargetLanguage.addItem(u"繁体中文", u"cht")
        self.comboTargetLanguage.addItem(u"越南语", u"vie")

        self.labelSrcFile = QLabel(u"源文件")
        self.inputSrcFile = QLineEdit()
        self.inputSrcBtn = QPushButton(u"打开文件")

        self.connect(self.inputSrcBtn, SIGNAL(
            "clicked()"), self.onInputSrcBtnClicked)

        self.labelTargetFile = QLabel(u"保存文件")
        self.inputTargetFile = QLineEdit()
        self.inputTargetBtn = QPushButton(u"选择文件名")
        self.connect(self.inputTargetBtn, SIGNAL(
            "clicked()"), self.onInputTargetBtnClicked)

        self.cvtBtn = QPushButton(u"开始翻译")
        self.connect(self.cvtBtn, SIGNAL("clicked()"), self.onCvtBtnClicked)

        self.gridLayout = QGridLayout()

        self.gridLayout.addWidget(self.labelTargetLanguage, 0, 0)
        self.gridLayout.addWidget(self.comboTargetLanguage, 0, 1)

        self.gridLayout.addWidget(self.labelSrcFile, 1, 0)
        self.gridLayout.addWidget(self.inputSrcFile, 1, 1)
        self.gridLayout.addWidget(self.inputSrcBtn, 1, 2)

        self.gridLayout.addWidget(self.labelTargetFile, 2, 0)
        self.gridLayout.addWidget(self.inputTargetFile, 2, 1)
        self.gridLayout.addWidget(self.inputTargetBtn, 2, 2)

        self.vLayout = QVBoxLayout()
        self.vLayout.addLayout(self.gridLayout)
        self.vLayout.addWidget(self.cvtBtn)
        self.setLayout(self.vLayout)

    @pyqtSlot()
    def onInputSrcBtnClicked(self):
        fileName = QFileDialog.getOpenFileName(
            self, u"打开要翻译的文件", u".", u"*.ts")
        if fileName.isEmpty():
            QMessageBox.warning(self, u"警告", u"未选择要翻译的文件", QMessageBox.Ok)
            return

        self.inputSrcFile.setText(fileName)

    @pyqtSlot()
    def onInputTargetBtnClicked(self):
        fileName = QFileDialog.getSaveFileName(
            self, u"选择另存为的文件名", u".", u"*.ts")
        if fileName.isEmpty():
            QMessageBox.warning(self, u"警告", u"未选择要保存为的文明名", QMessageBox.Ok)
            return

        self.inputTargetFile.setText(fileName)

        
    def process_file(self, src, dst, lang):
        self.cvtBtn.setText(u"正在翻译中...")
        tree = ET.parse(src)

        root = tree.getroot()
        source = ""

        for ctx in root:
            for msg in ctx:
                if msg.tag == 'message':
                    for tag in msg:
                        if tag.tag == 'source':
                            source = tag.text
                        if tag.tag == 'translation':
                            tag.text = translate(source, lang)
                            print("source = {0}, translate = {1}".format(source, tag.text))
                            tag.set('type', '')
        tree.write(dst)

        self.cvtBtn.setText(u"开始翻译")

    @pyqtSlot()
    def onCvtBtnClicked(self):
        targetLang = self.comboTargetLanguage.itemData(
            self.comboTargetLanguage.currentIndex()).toString()
        srcFileName = self.inputSrcFile.text()
        if srcFileName.isEmpty():
            QMessageBox.warning(self, u"警告", u"未选择要翻译的文件", QMessageBox.Ok)
            return

        targetFileName = self.inputTargetFile.text()
        if targetFileName.isEmpty():
            QMessageBox.warning(self, u"警告", u"未选择要保存为的文明名", QMessageBox.Ok)
            return

        t = threading.Thread(target=self.process_file, args=(srcFileName, targetFileName, str(targetLang)))

        t.setDaemon(True)
        t.start()

if __name__ == "__main__":
    app = QApplication(sys.argv)

    widget = MainWidget()
    widget.show()
    
    app.exec_()
{% endhighlight %}
