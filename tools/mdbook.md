# mdbook文档

类似Gitbook，使用markdown来创建book。用起来挺方便的，准备用来作为本地markdown文档的管理。具体的用法看[官方文档](https://rust-lang.github.io/mdBook/)。

## 安装

这部分看官方文档就行。我用的是Windows版本的Precompiled binaries，直接下载就能用。

## 基本命令

### init

使用`mdbook init`初始化文档。结构如下：

```powershell
D:\Workspace\Document\notes\mdBook>mdbook init

Do you want a .gitignore to be created? (y/n)
y
What title would you like to give the book?
TestmdBook
2021-06-10 11:12:58 [INFO] (mdbook::book::init): Creating a new book with stub c
ontent

All done, no errors...

D:\Workspace\Document\notes\mdBook>tree /F
卷 LENOVO 的文件夹 PATH 列表
卷序列号为 2802-03F5
D:.
│  .gitignore
│  book.toml
│
├─book
└─src
        chapter_1.md
        SUMMARY.md
```

目录的话，`src`就是写markdown的地方。`book`就是渲染后的地方。SUMMARY.md是skeleton，整个项目的骨架，类似书本目录的存在。*（我试了下，book.toml的信息会覆盖，不过源文件倒是不会被修改，也还行）*

这里有个技巧，`init`会解析SUMMARY.md文件里的内容，如果列出来的不存在会自动创建，所以可以先从SUMMARY.md创建大纲。

也可以指定目录创建`mdbook init path/to/book`。

如果使用`--theme`，那么会把默认的主题拷贝到theme目录，这样就可以自己修改主题了。

### build

`build`用于生成book。生成book结构和`src`的一样。注意build会复制src目录下除了.md文件之外的所有文件。

同样可以指定book的根目录`mdbook build path/to/book`。

使用`--open, -o`在生成book后直接打开。

使用`--dest-dir, -d`修改默认生成的目录，默认目录在配置文件`book.toml`中`build.build-dir`字段，或者是`./book`。

### watch

我理解类似实时`build`，参数也和`build`一样。指定目录：`mdbook watch path/to/book`、`--open, -o`、`--dest-dir, -d`。

但是有一点要注意，`watch`不会渲染.gitignore中列出的文件和目录。.gitignore具体格式可以参考[gitignore documentation](https://git-scm.com/docs/gitignore)。而且只能是book根目录下的.gitignore，不是$HOME/.gitignore或者其他目录下的。

### serve

`mdbook serve`会启用一个简易的HTTP服务，默认是`localhost:3000`。和`watch`一样会实时处理目录中文件的变化，通过websocket实现。`mdbook serve path/to/book`、`--open, -o`、`--dest-dir, -d`的用法与`watch`命令类似。使用`mdbook serve path/to/book -p 8000 -n 127.0.0.1`指定监听IP和端口。当然也会受到.gitignore的影响。

### test

用于测试一些内容，当前只支持rustdoc的test，或者说当前只支持rust代码块。这个用的应该不多。

### clean

用于清除生成的内容。`mdbook clean path/to/book`、`--dest-dir (-d)`用法与上面类似。

## 格式

### SUMMARY.md

核心文件。Without this file, there is no book.还是结合官方样例来说明。这里用HTML的注释来描述。

```markdown
# Summary <!-- Title。可选、可修改、可省略，通常就用这个，不会被解析渲染。 -->

[A Prefix Chapter](prefix/prefix.md) <!-- Prefix Chapter。类似前言、导读。可选，但是如果用就得放到最前面 -->

# 第一部分 <!-- Part Title。可选，这里也是我自己加的。逻辑分割各章节，会被渲染成unclickable text -->

- [mdBook](README.md) <!-- Numbered Chapter。类似正式章节。可用-或者*，但是不能混用。 -->
- [Command Line Tool](cli/README.md)
    - [init](cli/init.md)
    - [build](cli/build.md)
    - [watch](cli/watch.md)
    - [serve](cli/serve.md)
    - [test](cli/test.md)
    - [clean](cli/clean.md)

# 第二部分

- [Format](format/README.md)
    - [SUMMARY.md](format/summary.md)
        - [Draft chapter]() <!-- Draft chapters。草稿，会被渲染成disabled links -->
    - [Configuration](format/configuration/README.md)
        - [General](format/configuration/general.md)
        - [Preprocessors](format/configuration/preprocessors.md)
        - [Renderers](format/configuration/renderers.md)
        - [Environment Variables](format/configuration/environment-variables.md)
    - [Theme](format/theme/README.md)
        - [index.hbs](format/theme/index-hbs.md)
        - [Syntax highlighting](format/theme/syntax-highlighting.md)
        - [Editor](format/theme/editor.md)
    - [MathJax Support](format/mathjax.md)
    - [mdBook-specific features](format/mdbook.md)
- [Continuous Integration](continuous-integration.md)
- [For Developers](for_developers/README.md)
    - [Preprocessors](for_developers/preprocessors.md)
    - [Alternative Backends](for_developers/backends.md)

----------- <!-- Separators。分割线。至少用3个--- -->

[Contributors](misc/contributors.md) <!-- Suffix Chapter。类似附录，要用就用到最后 -->
```

### 配置

主要是针对book.toml这个文件。还是以一个样例来说明。注意配置文件里的相对路径，都是以book的根目录，或者说book.toml文件所在位置为起点。

```toml
[book] # General metadata
title = "Example book" # 标题
author = "John Doe" # 作者
description = "The example book covers examples." # 描述。会被渲染成<head>的meta信息。
src = "src" # 指定源文件的位置。默认在book的根目录下的src目录里。
language = "en" # book的语言。会被渲染成<html lang="en">这种。

[rust] # Rust相关的配置，对test和playground有效。
edition = "2018"

[build] # build相关
build-dir = "my-example-book" # book渲染后的路径。默认是book根目录下的book/
create-missing = false # 是否创建SUMMARY.md列出的章节中不存在的文件。默认是true，false则会在渲染不存在文件的时候报错。
use-default-preprocessors = false # 是否使用默认预处理（links和index）。这个受到其他配置项的影响：
# 如果没有任何preprocessors配置声明，那么use-default-preprocessors将会生效，并且默认为true
# use-default-preprocessors = false 将禁用默认的preprocessors
# 如果有类似[preprocessor.links]之类的配置，那么将会无视use-default-preprocessors的配置，对应的preprocessors例如links将会启用。

# preprocessor还不是很懂，有些部分先把官方的内容复制过来，用的时候还是看官方文档吧。
[preprocessor.index] # 把README.md渲染成index.html。

[preprocessor.links] #  Expand the {{ #playground }}, {{ #include }}, and {{ #rustdoc_include }} handlebars helpers in a chapter to include the contents of a file.

[preprocessor.mathjax]
renderers = ["html"]  # mathjax only makes sense with the HTML renderer

[output.html]
theme = "my-theme" # 指定主题位置，否则使用默认主题
default-theme = "light" # Change Theme默认选择
preferred-dark-theme = "navy" # 浏览器启用Dark模式的主题
curly-quotes = true # 把引号从直的改成卷的
mathjax-support = false # 默认false
copy-fonts = true # 复制font.css和其他相关字体文件到输出目录
google-analytics = "UA-123456-7"
additional-css = ["custom.css", "custom2.css"]
additional-js = ["custom.js"]
no-section-label = false # 不显示章节数字
git-repository-url = "https://github.com/rust-lang/mdBook" # 都是github相关
git-repository-icon = "fa-github"
edit-url-template = "https://github.com/rust-lang/mdBook/edit/master/guide/{path}"
site-url = "/example-book/" # book的URL，默认是/
cname = "myproject.rs" # DNS相关
input-404 = "not-found.md" # 404页

[output.html.print] # 打印。这个挺好用的，可以把所有内容显示成一页。
enable = true

[output.html.fold]
enable = false
level = 0

[output.html.playground]
editable = false
copy-js = true
line-numbers = false

[output.html.search]
enable = true
limit-results = 30
teaser-word-count = 30 # 搜索结果预显示字数
use-boolean-and = true # 多个关键字搜索时，是否要求搜索结果必须包含所有关键字。默认false。
boost-title = 2 # 下面都是一些搜索相关结果权限提升的。
boost-hierarchy = 1
boost-paragraph = 1
expand = true # 类似模糊搜索，默认是true，搜索micro，那么microwave也会被匹配。
heading-split-level = 3
copy-js = true

[output.html.redirect]
"/appendices/bibliography.html" = "https://rustc-dev-guide.rust-lang.org/appendix/bibliography.html"
"/other-installation-methods.html" = "../infra/other-installation-methods.html"

[output.markdown] # 目前针对markdown renderer，只支持开启或者关闭。默认是关闭的。开启对test等命令比较有用。
```

还支持自定义的renderer，这个有需要就再看文档。

环境变量同样的用上再看文档。

### 主题

这个等到自己要定制主题的时候再看。这里先了解一些内容。

Highlight，使用的是highlight.js，所以可以自己替换。目前官网列出来支持的语言是：

> apache, armasm, bash, c, coffeescript, cpp, csharp, css, d, diff, go, handlebars, haskell, http, ini, java, javascript, json, julia, kotlin, less, lua, makefile, markdown, nginx, objectivec, perl, php, plaintext, properties, python, r, ruby, rust, scala, scss, shell, sql, swift, typescript, x86asm, xml, yaml

当然可以自己修改theme中的highlight.js和highlight.css来支持更多的语言高亮。

可以在代码块中使用#来隐藏某一行。

如果在playground配置了editable = true，可以在代码块中加入editable来启用编辑。还可以指定编辑器。

### MathJax

在[output.html]中配置mathjax-support = true，可以启用MathJax支持。但是分隔符和原来MathJax不太一样，官方说是后面会改。这个我还是建议自己编辑好在复制，不要直接解析。而且mathjax-support默认也是false。

### 其他功能

使用#隐藏代码块中的某行。

使用`{{#title My Title}}`修改章节的标题，这样可以与目录或者说SUMMARY.md中定义的不同。

使用`{{#include file.rs}}`引入文件。路径为相对路径。默认mdBook会把引入的文件解析成markdown，最常用的情况还是引入代码文件，所以常见格式是把文件引入到代码块中，防止被解析。

```
{{#include file.rs}}
```

引入文件的时候还可以引入特定行，例如：

- `{{#include file.rs:2}}`，引入第2行
- `{{#include file.rs::10}}`，引入前10行

- `{{#include file.rs:2:}}`，从第2行开始引入
- `{{#include file.rs:2:10}}`，引入第2行到第10行

看起来比较方便。但是也存在修改源文件导致引入失败的情况。mdBook还支持使用anchors来引入。满足`ANCHOR:\s*[\w_-]+`、`ANCHOR_END:\s*[\w_-]+`的格式即可。例如：

```rust
/* ANCHOR: all */

// ANCHOR: component
struct Paddle {
    hello: f32,
}
// ANCHOR_END: component

////////// ANCHOR: system
impl System for MySystem { ... }
////////// ANCHOR_END: system

/* ANCHOR_END: all */
```

那么引入的时候，就可以：

Here is a component:
```rust,no_run,noplayground
{{#include file.rs:component}}
```

Here is a system:
```rust,no_run,noplayground
{{#include file.rs:system}}
```

This is the full file.
```rust,no_run,noplayground
{{#include file.rs:all}}
```

当然anchor表达式那行不会被引入。

还有一个rustdoc_include，是用来引入rust代码，但是只显示指定行。有点类似把整个源文件都引入，但是除了指定行之外其他行都被#隐藏了。这个感觉对自己用处也不大。

当然还有playground引入，这个就是引入一个可执行的rust程序。上面的都得在mdbook test时才管用。

## 其他

持续集成，部署到Github或者Gitlab，这个用到再说。

preprocessors的开发，这个也是用到再说。

backends，这里官方文档给出了一个wordcount的示例，同样用到再说。

