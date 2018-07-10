---
title: GIT中关于换行符CRLF和LF的探究
date: 2018-07-10 21:06:57
tags:
 - JavaScript
 - 知识积累
---

平时在使用`hexo`写博客，提交发布的时候，总会在命令行报`warning`，大量的`CRLF`、`LF`、`CR`等字眼，而在使用`WebStorm`开发项目时，右下角除了编码模式，分支切换，行号等提示外，总会不经意间扫到`CRLF`的配置，点开看看还有`LF - Unix and OS X(\n)`和`CR - CLassic Mac(\r)`这样的配置项。本来以为是键盘按键的设置（win 和 mac有几个键不一样），今天随手一搜，发现原来是另外一回事儿，花半小时简单总结一下，写篇小博客。
<!-- more -->

### 背景

`CR`(Carriage Return)代表的是回车，使用`\r`来表示；`LF`（Line Feed）代表换行，使用`\n`来表示，由于历史的原因，各操作系统的文本换行符号是不一致的，`windows`系统使用`CRLF`，即`\r\n`来表示换行，`Uninx`和近些年的`OS X`使用`LF`，即`\n`来表示换行，更早的`Mac`系统则使用`CR`，即`\r`来表示换行，不过这系统现在已经很少人用了。

现在问题来了，多人协作开发时，不可能保证每个人使用的系统一致，这就会导致换行编码的不同，虽然说“换行”在人眼看来没有区别，但是`git`面前可就不一样了，在推拉代码的过程中，如果换行符不一样，我是给你显示文件有修改呢还是直接默认没有任何更改？

好在`git`提供了一个“换行符自动转换”功能帮我们处理了这个问题。默认情况下，`git`在远程仓库保存代码使用`Unix风格`，即换行符统一使用`LF`模式（`\n`），在推-拉代码的过程中，则有以下的规则：

- 提交代码时，`git`会将文本文件中的换行符转换为`LF`模式，这个过程也叫`标准化过程`；
- 拉取代码时，`git`会将试图将仓库中的代码转换为`CRLF`模式，这个过程也叫`转换`；

通过这样的`拉-推`自动转换，`git`不仅保持了远程仓库代码的一致性（Unix风格），而且保证了本地文件的兼容性（Windows系统）。

### GIT个性化配置

既然有默认配置，那就有个性化的选择，`git`提供了`core.autocrlf`和`core.safecrlf`两个配置项来供开发者自由配置。两者都支持`全局`、`本地`设置。

```c
git config --global core.autocrlf [true | input | false]   #全局设置
git config --local core.autocrlf [true | input | false]    #针对本项目设置

git config --global core.safecrlf [true | warn | false]    #全局设置
git config --local core.safecrlf [true | warn | false]     #针对本项目设置
```

#### core.autocrlf

`core.autocrlf`有三个值，默认是 `true`。这个配置用来决定要不要转换，怎么转换。

```javascript
// 默认设置，拉取时转换为CRLF，提交时转换为LF
git config --global core.autocrlf true

// 拉取时不转换，提交时转换为LF
git config --global core.autocrlf input

// 拉取时不转换，提交时也不转换
git config --global core.autocrlf false
```

#### core.safecrlf

`core.safecrlf`也有三个值，默认是 `false`。这个配置用来决定`add`代码时是否禁止提交混合换行符的文本文件。

```javascript
// 禁止提交混合换行符的文本文件(git add 的时候会被拦截，提示异常)
git config --global core.safecrlf true

// 提交混合换行符的文本文件的时候发出警告，但是不会阻止 git add 操作
git config --global core.safecrlf warn

//  默认配置，不禁止提交混合换行符的文本文件
git config --global core.safecrlf false
```

### 延深知识

上面的两个配置文件都是个人在本机的个性化设置，如果对于比较大的项目，多人协作的情况下不可能让每个人都去更改一下自己的配置，这个时候我们就可以在项目的根目录下添加一个`.gitattributes`文件，它的优先级高于`core.autocrlf`配置，它类似于 `.gitignore` 文件，随提交修改生效，一个项目中可以维持一份相同的配置。 当然，`.gitattributes`文件可不只有配置换行符，还有一些其他的复杂配置。 具体更深入的了解就等以后遇到了再去探究吧。


