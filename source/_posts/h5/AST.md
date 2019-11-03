---
title: AST
abbrlink: 7389a59f
date: 2019-11-02 16:57:32
tags:
    - h5
    - AST
---
## 前言
提起 AST 抽象语法树，大家可能并不感冒，认为它和你的领域没有多少交集。但是提到它的使用场景，也许会让你大吃一惊。原来它一直在你左右与你相伴，而你却不知。
## 简介
在计算机科学中，抽象语法树（abstract syntax tree 或者缩写为 AST），或者语法树（syntax tree），是源代码的抽象语法结构的树状表现形式，这里特指编程语言的源代码，例如less、sass、ES6、typescript、js等源代码。树上的每个节点都表示源代码中的一种结构。之所以说语法是「抽象」的，是因为这里的语法并不会表示出真实语法中出现的每个细节。
## 作用
前面说AST树一直伴随着我们左右，其实只要涉及到词法分析或语法分析的都会使用它，例如各种代码编辑、编译器，代码转换器（工具）等，而在前端中，我们接触到的功能主要有以下几种：
- JS反编译，语法分析
- Babel编译ES6语法，Babel插件
- 代码高亮
- 关键字匹配
- 作用域判断
- 代码压缩
- 代码转换
- 代码补全
- etc...

## AST Explorer
由于语法树一直以来比较抽象，没有一个比较好的方式去学习了解它，我们通常了解语法树的结构是通过[AST Explorer](https://astexplorer.net/)去查看相关的树结构。其中几乎支持所有前端语法，除部分预编译语言，如less、sass、stylus等。
通过该解释器，我们先观察一个简单的ES6代码：
```javascript
let messages=["I'm a student!"];
```
生成的JSON格式如下：
```json5
{
  "type": "Program",
  "start": 0,
  "end": 32,
  "body": [
    {
      "type": "VariableDeclaration",
      "start": 0,
      "end": 32,
      "declarations": [
        {
          "type": "VariableDeclarator",
          "start": 4,
          "end": 31,
          "id": {
            "type": "Identifier",
            "start": 4,
            "end": 12,
            "name": "messages"
          },
          "init": {
            "type": "ArrayExpression",
            "start": 13,
            "end": 31,
            "elements": [
              {
                "type": "Literal",
                "start": 14,
                "end": 30,
                "value": "I'm a student!",
                "raw": "\"I'm a student!\""
              }
            ]
          }
        }
      ],
      "kind": "let"
    }
  ],
  "sourceType": "module"
}
```
而他的语法树如下：
{% asset_img img-left ex_1.png ast1 %}
我们发现AST语法树和DOM树很类似，都是树形结构，结构层次清清楚楚，再看一个`expression`的例子：

