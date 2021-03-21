---
title: 用PyQT写小工具
tags:
  - pyqt5
  - QTdesigner
summary: 使用PyQT5时遇到的一些坑
abbrlink: '3515'
date: 2021-01-19 10:24:29
---

最近写的论文有手动建立案例的需求，一开始采用的方法是建一个json模板，然后在模板上改关键词，就像这样：

```json
{
    "有限元分析案例": [
        {
            "产品信息": [
                {
                    "产品名称": [
                        "none"
                    ],
                    "分析对象": [
                        "none"
                    ],
                    "产品材料": [
                        "none"
                    ]
                }
            ],
            "设计": [
                {
                    "设计尺寸": [
                        "none"
                    ],
                    "设计参数": [
                        "none"
                    ],
                    "补强条件": [
                        "none"
                    ]
                }
            ],
            "工况": [
                {
                    "介质": [
                        "none"
                    ],
                    "环境条件": [
                        "none"
                    ],
                    "应力条件": [
                        "none"
                    ]
                }
            ],
            "分析条件": [
                {
                    "材料属性": [
                        "none"
                    ],
                    "边界条件": [
                        "none"
                    ],
                    "接触类型": [
                        "none"
                    ],
                    "约束条件": [
                        "none"
                    ],
                    "载荷类型": [
                        "none"
                    ]
                }
            ],
            "分析类型": [
                "none"
            ]
        }
    ]
}
```

但是这样做效率太低了，遂有了做一个小工具的想法。主要实现几个功能：

+ 信息通过文本框录入，是最基本的交互方式；
+ 建立的案例以json格式保存；
+ 要能够读取和修改案例，实现案例的管理（还没做😂）

最后做出来这个样子的界面：

![小工具的主界面](https://my-picbed.oss-cn-hangzhou.aliyuncs.com/img/20210119103800.png)

总结一下做这个的时候遇到的一些坑：

### 1. Text与PlainText（小坑）

QTdesigner中有两种文本框输入控件

![两种控件](https://my-picbed.oss-cn-hangzhou.aliyuncs.com/img/20210119104145.png)

一开始我用了第一种`Text Edit`，手动输入文本没有问题，但是从别处复制粘贴过来的文本都是带格式的。再看这俩控件的图标，原来`Plain Text`意思是无论输入什么格式的文本，都会变成不带格式的普通文本。

### 2. Layout和Frame（小坑）

一开始写的时候标签和文本框都是手动对齐的😂

直到用上了`Layout`和`Frame`，这俩能让你的图形界面看起来更像一个图形界面。但是这俩有啥不同能，看看他们之间参数上的差异：

![Layout](https://my-picbed.oss-cn-hangzhou.aliyuncs.com/img/20210119105537.png)

![Frame](https://my-picbed.oss-cn-hangzhou.aliyuncs.com/img/20210119105653.png)

简单来说，`Layout`作为幕后黑手，规定了整个图形界面的布局样式，我们看不到他。而`Frame`作为容器，我们可以通过改变他的样式来给`Frame`之间做区分。

### 3. Table Widget（大坑）

因为一个产品中可能会有好几种材料，这些材料要考虑的参数也不一样。所以要一条一条添加材料的信息。一开始的想法是，点一次按钮就往Frame里加一个这样的Widget：

![材料信息Widget](https://my-picbed.oss-cn-hangzhou.aliyuncs.com/img/20210119111036.png)

可这就带来了一个新的问题，别的文本框都能这样获取文本

```python
CaseName = self.CaseName.toPlainText()
```

但是这个Widget中文本框没有标签名，需要遍历整个Frame来获取文本。这很麻烦，效率很低。

然后我注意到了QTDesigner中的Table Widget，该控件更适合这种场景。但又遇到一个问题，如果我要初始化表格，需要一行一行的删除，正常的思路是从头删到尾，可怎么写都会出现，有一行没删掉或者多删了一行的问题。找了一下网上的方法，原来要从最后一行开始，写一个逆循环来删：

```python
	def initTable(self):
		"""
		初始化材料表格
		"""
		for i in range(0, self.MaterialTable.rowCount())[::-1]:  # 删除新增的行
			self.MaterialTable.removeRow(i)

		self.MaterialTable.insertRow(0)
```

