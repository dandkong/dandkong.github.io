# PuerTS学习路线


## 前言

最近看到腾讯又出了个游戏脚本的热更方案，相对lua有以下的优势。

> JavaScript生态有众多的库和工具链，结合专业商业引擎的渲染能力，快速打造游戏
>
> 相比游戏领域常用的lua脚本，TypeScript的静态类型检查有助于编写更健壮，可维护性更好的程序

## 安装

[安装PuerTS | PUER Typescript](https://puerts.github.io/docs/puerts/unity/install/)

安装包到Unity，在合适的地方接入JSEnv

## 自定义加载器

PuerTS自带一个加载器，通过Resource加载JS代码，但是实际开发的时候推荐的是TypeScript，需要转译到JS再执行。

编辑器中推荐使用TS-Loader，编辑器中直接使用TS进行开发，没有其他需要关注的地方。

> 提供一个PuerTS的Loader，使你在Editor下，可以直接读取TS。
>
> 无需研究tsconfig、无需研究ESM、CommonJS，无需自行编译ts，无需理会和调试相关的debugpath/sourceMap/控制台跳转。

https://github.com/zombieyang/puerts-ts-loader

编辑器中直接使用TS开发，在发布时还需要编译成JS，参考ts-loader中的TSReleaser-Resources.cs函数，在发布资源前编译成JS。

编译好的JS要支持热更，所以使用默认的加载器，自己定义一个加载器，传入的参数是JS的路径，返回的是JS的代码内容，结合项目的资源加载返回即可，我的Demo使用了YooAssets框架。

## 调试

用类似NodeJs的方式调试即可。

[VSCode Debug 指引 | PUER Typescript](https://puerts.github.io/docs/puerts/unity/knowjs/debugging)

## TS学习

TS拥有静态类型检测，能编写更具有健壮性的代码，也有更多现有的资源。

[深入理解 TypeScript | 深入理解 TypeScript](https://jkchao.github.io/typescript-book-chinese/#why)

[Handbook - The TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)

[TypeScript 入门教程](https://ts.xcatliu.com/)

[ES6 入门教程](https://es6.ruanyifeng.com/)

[前言 | TypeScript手册](https://bosens-china.github.io/Typescript-manual/describe/)

