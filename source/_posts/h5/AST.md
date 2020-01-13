---
title: AST
abbrlink: 7389a59f
date: 2019-11-02 16:57:32
categories:
    - h5
tags:
    - AST
    - Babel
top: true
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
我们发现AST语法树和DOM树很类似，都是树形结构，结构层次清清楚楚。

<hr>

再看一个`expression`的例子：
```javascript
(1+2)*3
```
JSON格式如下：
```json5
{
  "type": "Program",
  "start": 0,
  "end": 7,
  "body": [
    {
      "type": "ExpressionStatement",
      "start": 0,
      "end": 7,
      "expression": {
        "type": "BinaryExpression",
        "start": 0,
        "end": 7,
        "left": {
          "type": "BinaryExpression",
          "start": 1,
          "end": 4,
          "left": {
            "type": "Literal",
            "start": 1,
            "end": 2,
            "value": 1,
            "raw": "1"
          },
          "operator": "+",
          "right": {
            "type": "Literal",
            "start": 3,
            "end": 4,
            "value": 2,
            "raw": "2"
          }
        },
        "operator": "*",
        "right": {
          "type": "Literal",
          "start": 6,
          "end": 7,
          "value": 3,
          "raw": "3"
        }
      }
    }
  ],
  "sourceType": "module"
}
```
对应的AST树如下：
{% asset_img img-left ex_2.png ast2 %}
将表达式的`()`删除，我们做个对比：
```json5
{
  "type": "Program",
  "start": 0,
  "end": 5,
  "body": [
    {
      "type": "ExpressionStatement",
      "start": 0,
      "end": 5,
      "expression": {
        "type": "BinaryExpression",
        "start": 0,
        "end": 5,
        "left": {
          "type": "Literal",
          "start": 0,
          "end": 1,
          "value": 1,
          "raw": "1"
        },
        "operator": "+",
        "right": {
          "type": "BinaryExpression",
          "start": 2,
          "end": 5,
          "left": {
            "type": "Literal",
            "start": 2,
            "end": 3,
            "value": 2,
            "raw": "2"
          },
          "operator": "*",
          "right": {
            "type": "Literal",
            "start": 4,
            "end": 5,
            "value": 3,
            "raw": "3"
          }
        }
      }
    }
  ],
  "sourceType": "module"
}
```
通过对比发现，括号限制了表达式的优先级，在AST树中影响的是left和right中的结构顺序：
1. 在确定类型为`ExpressionStatement`后，它会按照代码执行的先后顺序，将表达式`BinaryExpression`分为`left`，`operator` 和 `right` 三块
2. `left`：表示一个运算符的左侧，可以是另一个表达式也可以是一个具体的值
3. `right`：表示一个运算符的右侧，可以是另一个表达式也可以是一个具体的值
4. `left`和`right`都标明了类型，起止位置，值等信息；
5. `operator`定义操作运算符；

通过这样的结构我们可以自己实现复杂的`expression`的AST树，例如`(a*b+c)/d`：
```json5
{
  "type": "Program",
  "start": 0,
  "end": 9,
  "body": [
    {
      "type": "ExpressionStatement",
      "start": 0,
      "end": 9,
      "expression": {
        "type": "BinaryExpression",
        "start": 0,
        "end": 9,
        "left": {
          "type": "BinaryExpression",
          "start": 1,
          "end": 6,
          "left": {
            "type": "BinaryExpression",
            "start": 1,
            "end": 4,
            "left": {
              "type": "Identifier",
              "start": 1,
              "end": 2,
              "name": "a"
            },
            "operator": "*",
            "right": {
              "type": "Identifier",
              "start": 3,
              "end": 4,
              "name": "b"
            }
          },
          "operator": "+",
          "right": {
            "type": "Identifier",
            "start": 5,
            "end": 6,
            "name": "c"
          }
        },
        "operator": "/",
        "right": {
          "type": "Identifier",
          "start": 8,
          "end": 9,
          "name": "d"
        }
      }
    }
  ],
  "sourceType": "module"
}
```

<hr>

再来看看我们常用的箭头函数的AST树：
```javascript
const test = (a,b) => {
  return a+b;
}
```
JSON格式如下：
```json5
{
  "type": "Program",
  "start": 0,
  "end": 39,
  "body": [
    {
      "type": "VariableDeclaration",
      "start": 0,
      "end": 39,
      "declarations": [
        {
          "type": "VariableDeclarator",
          "start": 6,
          "end": 39,
          "id": {
            "type": "Identifier",
            "start": 6,
            "end": 10,
            "name": "test"
          },
          "init": {
            "type": "ArrowFunctionExpression",
            "start": 13,
            "end": 39,
            "id": null,
            "expression": false,
            "generator": false,
            "async": false,
            "params": [
              {
                "type": "Identifier",
                "start": 14,
                "end": 15,
                "name": "a"
              },
              {
                "type": "Identifier",
                "start": 16,
                "end": 17,
                "name": "b"
              }
            ],
            "body": {
              "type": "BlockStatement",
              "start": 22,
              "end": 39,
              "body": [
                {
                  "type": "ReturnStatement",
                  "start": 26,
                  "end": 37,
                  "argument": {
                    "type": "BinaryExpression",
                    "start": 33,
                    "end": 36,
                    "left": {
                      "type": "Identifier",
                      "start": 33,
                      "end": 34,
                      "name": "a"
                    },
                    "operator": "+",
                    "right": {
                      "type": "Identifier",
                      "start": 35,
                      "end": 36,
                      "name": "b"
                    }
                  }
                }
              ]
            }
          }
        }
      ],
      "kind": "const"
    }
  ],
  "sourceType": "module"
}
```
对应的AST结构如下：
{% asset_img ex_3.png ast3 %}
我们会发现，其中相对于上面的简单例子，又增加了几个新的类型：
- ArrowFunctionExpression
- BlockStatement
- ReturnStatement
到这里，其实我们已经慢慢明白了：
抽象语法树其实就是将一类标签转化成通用标识符，从而结构出的一个类似于树形结构的语法树，只要了解其语法树的格式，类型定义就能深入的理解AST。

## 深入原理
AST Explorer 可视化工具能够让我们快速认知AST，那么这些可视化工具、编译器、转换插件内部是怎么实现的呢，我们来看一个例子：
```javascript
function getAST(){}
```
其JSON格式如下：
```json5
{
  "type": "Program",
  "start": 0,
  "end": 19,
  "body": [
    {
      "type": "FunctionDeclaration",
      "start": 0,
      "end": 19,
      "id": {
        "type": "Identifier",
        "start": 9,
        "end": 15,
        "name": "getAST"
      },
      "expression": false,
      "generator": false,
      "async": false,
      "params": [],
      "body": {
        "type": "BlockStatement",
        "start": 17,
        "end": 19,
        "body": []
      }
    }
  ],
  "sourceType": "module"
}
```
AST如下：
{% asset_img ex_4.png ast4 %}
我们通过第三方模块`esprima`、`estraverse`、`escodegen`三个包来实现树的遍历
- `esprima` : 一个将js代码解析成AST树的包
- `estraverse`：遍历树的包
- `escodegen`：生成新的AST树的包
```typescript
const esprima = require('esprima'); //解析js的语法的包
const estraverse = require('estraverse'); //遍历树的包
const escodegen = require('escodegen'); //生成新的树的包
let code = `function getAST(){}`;
//解析js的语法
let tree = esprima.parseScript(code);
//遍历树
estraverse.traverse(tree, {
  enter(node) {
    console.log('enter: ' + node.type);
  },
  leave(node) {
    console.log('leave: ' + node.type);
  }
});
//生成新的树
let r = escodegen.generate(tree);
console.log(r);
```
运行后输出结果为：
```
enter: Program
enter: FunctionDeclaration
enter: Identifier
leave: Identifier
enter: BlockStatement
leave: BlockStatement
leave: FunctionDeclaration
leave: Program
function getAST() {
}
```
可以发现遍历过程使用的是深度优先算法，当然我们也可以自己实现对应的遍历算方法，不过遍历算法需要支持所有的类型，否则遍历会出现问题，除非你仅需要针对特定类型进行处理，例如`postcss`中的提供的遍历API。
通过遍历我们可以实现一定的代码转换功能，例如将上述函数名修改为`newAST`：
```typescript
const esprima = require('esprima'); //解析js的语法的包
const estraverse = require('estraverse'); //遍历树的包
const escodegen = require('escodegen'); //生成新的树的包
let code = `function getAST(){}`;
//解析js的语法
let tree = esprima.parseScript(code);
//遍历树
estraverse.traverse(tree, {
  enter(node) {
    console.log('enter: ' + node.type);
    if (node.type === 'Identifier') {
      node.name = 'newAST';
    }
  }
});
//生成新的树
let r = escodegen.generate(tree);
console.log(r);
```
运行后结果为：
```
enter: Program
enter: FunctionDeclaration
enter: Identifier
enter: BlockStatement
function newAST() {
}
```
可以看到，在我们的干预下，输出的结果发生了变化，方法名编译后方法名变成了newAST。
这就是抽象语法树的强大之处，本质上通过编译，我们可以去改变任何输出结果。
这也是大部分开发工具编译的核心，包括babel、less、sass、postcss前端常用转换工具等。

<hr>

AST Node类型大致有以下集合：
```typescript
type INodeType= "Identifier" | "SimpleLiteral" | "RegExpLiteral" |
 "Program" | "FunctionDeclaration" | "FunctionExpression" |
 "ArrowFunctionExpression" | "SwitchCase" | "CatchClause" |
 "VariableDeclarator" | "ExpressionStatement" | "BlockStatement" |
 "EmptyStatement" | "DebuggerStatement" | "WithStatement" |
 "ReturnStatement" | "LabeledStatement" | "BreakStatement" |
 "ContinueStatement" | "IfStatement" | "SwitchStatement" |
 "ThrowStatement" | "TryStatement" | "WhileStatement" |
 "DoWhileStatement" | "ForStatement" | "ForInStatement" |
 "ForOfStatement" | "VariableDeclaration" | "ClassDeclaration" |
 "ThisExpression" | "ArrayExpression" | "ObjectExpression" |
 "YieldExpression" | "UnaryExpression" | "UpdateExpression" |
 "BinaryExpression" | "AssignmentExpression" | "LogicalExpression" |
 "MemberExpression" | "ConditionalExpression" |
 "SimpleCallExpression" | "NewExpression" | "SequenceExpression" |
 "TemplateLiteral" | "TaggedTemplateExpression" | "ClassExpression" |
 "MetaProperty" | "AwaitExpression" | "Property" |
 "AssignmentProperty" | "Super" | "TemplateElement" | "SpreadElement" |
 "ObjectPattern" | "ArrayPattern" | "RestElement" | "AssignmentPattern" |
 "ClassBody" | "MethodDefinition" | "ImportDeclaration" | 
 "ExportNamedDeclaration" | "ExportDefaultDeclaration" |
 "ExportAllDeclaration" | "ImportSpecifier" | "ImportDefaultSpecifier" |
 "ImportNamespaceSpecifier" | "ExportSpecifier" | "Directive" | "DirectiveLiteral" |
 "CallExpression" | "File" | "StringLiteral" | "NumericLiteral" | "NullLiteral" |
 "BooleanLiteral" | "ObjectMethod" | "ObjectProperty" | "ExportNamedDeclaration" |
 "ClassMethod" | "AnyTypeAnnotation" | "ArrayTypeAnnotation" |
 "BooleanTypeAnnotation" | "BooleanLiteralTypeAnnotation" |
 "NullLiteralTypeAnnotation" | "ClassImplements" | "ClassProperty" | "DeclareClass" |
 "DeclareFunction" | "DeclareInterface" | "DeclareModule" | "DeclareTypeAlias" |
 "DeclareVariable" | "ExistentialTypeParam" | "FunctionTypeAnnotation" |
 "FunctionTypeParam" | "GenericTypeAnnotation" | "InterfaceExtends" | "InterfaceDeclaration" |
 "IntersectionTypeAnnotation" | "MixedTypeAnnotation" | "NullableTypeAnnotation" |
 "NumericLiteralTypeAnnotation" | "NumberTypeAnnotation" | "StringLiteralTypeAnnotation" |
 "StringTypeAnnotation" | "ThisTypeAnnotation" | "TupleTypeAnnotation" |
 "TypeofTypeAnnotation" | "TypeAlias" | "TypeAnnotation" | "TypeCastExpression" |
 "TypeParameterDeclaration" | "TypeParameterInstantiation" | "ObjectTypeAnnotation" |
 "ObjectTypeCallProperty" | "ObjectTypeIndexer" | "ObjectTypeProperty" |
 "QualifiedTypeIdentifier" | "UnionTypeAnnotation" | "VoidTypeAnnotation" | "JSXAttribute" |
 "JSXClosingElement" | "JSXElement" | "JSXEmptyExpression" | "JSXExpressionContainer" |
 "JSXIdentifier" | "JSXMemberExpression" | "JSXNamespacedName" | "JSXOpeningElement" |
 "JSXSpreadAttribute" | "JSXText" | "Noop" | "ParenthesizedExpression" | "AwaitExpression" |
 "BindExpression" | "Decorator" | "DoExpression" | "ExportDefaultSpecifier" |
 "ExportNamespaceSpecifier" | "RestProperty" | "SpreadProperty" | "TSAnyKeyword" |
 "TSArrayType" | "TSAsExpression" | "TSBooleanKeyword" | "TSCallSignatureDeclaration" |
 "TSConstructSignatureDeclaration" | "TSConstructorType" | "TSDeclareFunction" |
 "TSDeclareMethod" | "TSEnumDeclaration" | "TSEnumMember" | "TSExportAssignment" |
 "TSExpressionWithTypeArguments" | "TSExternalModuleReference" | "TSFunctionType" |
 "TSImportEqualsDeclaration" | "TSIndexSignature" | "TSIndexedAccessType" | "TSInterfaceBody" |
 "TSInterfaceDeclaration" | "TSIntersectionType" | "TSLiteralType" | "TSMappedType" |
 "TSMethodSignature" | "TSModuleBlock" | "TSModuleDeclaration" | "TSNamespaceExportDeclaration" |
 "TSNeverKeyword" | "TSNonNullExpression" | "TSNullKeyword" | "TSNumberKeyword" |
 "TSObjectKeyword" | "TSParameterProperty" | "TSParenthesizedType" | "TSPropertySignature" |
 "TSQualifiedName" | "TSStringKeyword" | "TSSymbolKeyword" | "TSThisType" | "TSTupleType" |
 "TSTypeAliasDeclaration" | "TSTypeAnnotation" | "TSTypeAssertion" | "TSTypeLiteral" |
 "TSTypeOperator" | "TSTypeParameter" | "TSTypeParameterDeclaration" |
 "TSTypeParameterInstantiation" | "TSTypePredicate" | "TSTypeQuery" | "TSTypeReference" |
 "TSUndefinedKeyword" | "TSUnionType" | "TSVoidKeyword";
```
其中包括JSX、typescript 的AST类型，基本上根据类型名称都能了解其表示的含义，AST树就是由这些类型及其特定的属性组成的一个层级结构。
介绍到这边，你应该联想到 `Babel`，联想到 `js 混淆`，联想到更多背后的东西。接下来，我们要介绍介绍 `Babel` 是如何将 `ES6` 转成 `ES5` 的。

## Babel
Babel本质上是一个编译器，他是将很多超前的，未在W3C规范中的语法编译转换成复合W3C规范的，旧代码执行器支持的代码。其具体内容本文不作介绍，如需了解请查看{% post_link h5/Babel Babel %}。
由于Es6、ES7、ES8、ES9等兼容性问题，在旧版本的代码执行器（浏览器环境）下，无法正确识别并执行，因此我们前端开发通常在build时使用babel将其转换成ES5的代码，确保低版本兼容性。下面我们来看看Babel是如何工作的。
箭头函数转成function：
```typescript
let sum = (a, b)=>{return a+b};
```
JSON格式如下：
```json5
{
  "type": "Program",
  "start": 0,
  "end": 31,
  "body": [
    {
      "type": "VariableDeclaration",
      "start": 0,
      "end": 31,
      "declarations": [
        {
          "type": "VariableDeclarator",
          "start": 4,
          "end": 30,
          "id": {
            "type": "Identifier",
            "start": 4,
            "end": 7,
            "name": "sum"
          },
          "init": {
            "type": "ArrowFunctionExpression",
            "start": 10,
            "end": 30,
            "id": null,
            "expression": false,
            "generator": false,
            "async": false,
            "params": [
              {
                "type": "Identifier",
                "start": 11,
                "end": 12,
                "name": "a"
              },
              {
                "type": "Identifier",
                "start": 14,
                "end": 15,
                "name": "b"
              }
            ],
            "body": {
              "type": "BlockStatement",
              "start": 18,
              "end": 30,
              "body": [
                {
                  "type": "ReturnStatement",
                  "start": 19,
                  "end": 29,
                  "argument": {
                    "type": "BinaryExpression",
                    "start": 26,
                    "end": 29,
                    "left": {
                      "type": "Identifier",
                      "start": 26,
                      "end": 27,
                      "name": "a"
                    },
                    "operator": "+",
                    "right": {
                      "type": "Identifier",
                      "start": 28,
                      "end": 29,
                      "name": "b"
                    }
                  }
                }
              ]
            }
          }
        }
      ],
      "kind": "let"
    }
  ],
  "sourceType": "module"
}
```
我们通过Babel模拟转换过程如下：
```typescript
const babel = require('babel-core'); //babel核心解析库
const t = require('babel-types'); //babel类型转化库
let code = `let sum = (a, b)=>{return a+b}`;
let ArrowPlugins = {
//访问者模式
visitor: {
  //捕获匹配的API
    ArrowFunctionExpression(path) {
      let { node } = path;
      let body = node.body;
      let params = node.params;
      let r = t.functionExpression(null, params, body, false, false);
      path.replaceWith(r);
    }
  }
}
let d = babel.transform(code, {
  plugins: [
    ArrowPlugins
  ]
})
console.log(d.code);
```
`babel-core`是Babel的核心库，`babel-types`是Babel的类型转换库，通常我们开发Babel插件时需要了解这两个库，此处不作详细介绍，通过运行上述代码，我们得到的结果是：
```typescript
let sum = function (a, b) {
  return a + b;
};
```
这里，我们完美的将箭头函数转换成了标准函数。`但是这也仅仅是针对这一一种情况而言，箭头函数还支持简写，如果是简写那么，该插件就会报错，因为其body中的结构和之前不一样了，此时JSON格式如下：`
```json5
{
  "type": "Program",
  "start": 0,
  "end": 21,
  "body": [
    {
      "type": "VariableDeclaration",
      "start": 0,
      "end": 21,
      "declarations": [
        {
          "type": "VariableDeclarator",
          "start": 4,
          "end": 21,
          "id": {
            "type": "Identifier",
            "start": 4,
            "end": 7,
            "name": "sum"
          },
          "init": {
            "type": "ArrowFunctionExpression",
            "start": 10,
            "end": 21,
            "id": null,
            "expression": true,
            "generator": false,
            "async": false,
            "params": [
              {
                "type": "Identifier",
                "start": 11,
                "end": 12,
                "name": "a"
              },
              {
                "type": "Identifier",
                "start": 14,
                "end": 15,
                "name": "b"
              }
            ],
            "body": {
              "type": "BinaryExpression",
              "start": 18,
              "end": 21,
              "left": {
                "type": "Identifier",
                "start": 18,
                "end": 19,
                "name": "a"
              },
              "operator": "+",
              "right": {
                "type": "Identifier",
                "start": 20,
                "end": 21,
                "name": "b"
              }
            }
          }
        }
      ],
      "kind": "let"
    }
  ],
  "sourceType": "module"
}
```
很明显，body 类型变成 BinaryExpression 不再是 BlockStatement，所以需要做一些兼容修改：
```typescript
const babel = require('babel-core'); //babel核心解析库
const t = require('babel-types'); //babel类型转化库
let code = `let sum = (a, b)=> a+b`;
let ArrowPlugins = {
//访问者模式
  visitor: {
  //捕获匹配的API
    ArrowFunctionExpression(path) {
      let { node } = path;
      let params = node.params;
      let body = node.body;
      if(!t.isBlockStatement(body)){
        let returnStatement = t.returnStatement(body);
        body = t.blockStatement([returnStatement]);
      }
      let r = t.functionExpression(null, params, body, false, false);
      path.replaceWith(r);
    }
  }
}
let d = babel.transform(code, {
  plugins: [
    ArrowPlugins
  ]
})
console.log(d.code);
```
最终输出结果：
```typescript
let sum = function (a, b) {
    return a + b;
};
```
到这边，看起来这个插件已经完美了，但是真实的[babel-plugin-transform-arrow-function](https://babeljs.io/docs/en/babel-plugin-transform-arrow-functions)并不是这么简单，它内部还涉及到this指向的问题，此处不作详细讲解。

<hr>

上文我们简单演示了 Babel 是如何来编译代码的，但是并非简单如此。
Babel 使用一个基于 ESTree 并修改过的 AST，其使用的是开源的[babylon](https://github.com/babel/babylon)解析库，目前改名为[babel-parser](https://github.com/babel/babel/tree/master/packages/babel-parser)。
正如我们上面示例代码一样，Babel 的三个主要处理步骤分别是： 解析（parse），转换（transform），生成（generate）。
1. 解析（parse）：解析步骤接收代码并输出处理后的 AST。 这个步骤分为两个阶段：词法分析 Lexical Analysis 和语法分析Syntactic Analysis。
   - 词法分析：词法分析阶段把字符串形式的代码转换为令牌（tokens） 流。你可以把令牌看作是一个扁平的语法片段数组：
   ```javascript
    n * n;
   ```
   例如上面的代码片段，解析结果如下：
   ```javascript
    [
      { type: { ... }, value: "n", start: 0, end: 1, loc: { ... } },
      { type: { ... }, value: "*", start: 2, end: 3, loc: { ... } },
      { type: { ... }, value: "n", start: 4, end: 5, loc: { ... } },
      ...
    ]
   ```
   每一个 type 有一组属性来描述该令牌，和 AST 节点一样它们也有 start，end，loc 属性：
   ```json5
   {
     type: {
       label: 'name',
       keyword: undefined,
       beforeExpr: false,
       startsExpr: true,
       rightAssociative: false,
       isLoop: false,
       isAssign: false,
       prefix: false,
       postfix: false,
       binop: null,
       updateContext: null
     },
     ...
   }
   ```
   - 语法分析：语法分析阶段会把一个令牌流转换成 AST 的形式。 这个阶段会使用令牌中的信息把它们转换成一个 AST 的表述结构，这样更易于后续的操作。

2. 转换（transform）：接收 AST 并对其进行遍历，在此过程中对节点进行添加、更新及移除等操作。 这是 Babel 或是其他编译器中最复杂的过程，同时也是插件将要介入工作的部分。
3. 生成（generate）：代码生成步骤把最终（经过一系列转换之后）的 AST 转换成字符串形式的代码，同时还会创建源码映射（source maps）。代码生成其实很简单：深度优先遍历整个 AST，然后构建可以表示转换后代码的字符串。

了解这这些过程，其实会发现我们上面实现的箭头函数转换插件变得很简单。

## 遍历
AST的处理和转换需要使用到树形的遍历，树形遍历算法通常有前序、中序、后序、广度优先、深度优先，关于树遍历的算法本文不作详解，有兴趣的可以查阅资料或者转到{% post_link h5/data-structure/tree Tree Structure %}中了解。
我们发现上述使用了很多关于树操作的API，这些API并不是AST本身自带的API，而是每个parser赋予它的，AST本质上是一个树形数据结构，也可以看做是一个JSON，具体的每个parser的API我们后面介绍相关插件开发的时候再做补充说明。

## 扩展（具体语法树）
AST被称为抽象语法树，看到这个名词的时候我们很可能会想，那对应的具体语法树呢？
对的，具体语法树是存在的，具体语法树通常被称作分析树。一般的，在源代码的翻译和编译过程中，语法分析器创建出分析树。一旦AST 被创建出来，在后续的处理过程中，比如语义分析阶段，会添加一些信息。

