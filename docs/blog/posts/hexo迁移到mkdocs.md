---
title: hexo迁移到mkdoc
date: 2024-01-07 21:47:39
tags:
  - hexo
  - mkdocs
  - cloudfare
  - obsidian
categories:
  - 折腾
---
我比较喜欢hexo的tags字云的效果，不过由于以下原因，我打算迁移到新的静态博客框架mkdocs material theme
- hexo nodejs的依赖实在是太麻烦了
- hexo展示的信息量太少了（也可能是我使用的模板的问题）
  - 我感觉这一点上mkdoc效果就很好，在大显示器上能够显示更多信息，查阅博客的效率更高

TODO
- [ ] 集成评论区:[Adding a comment system - Material for MkDocs (squidfunk.github.io)](https://squidfunk.github.io/mkdocs-material/setup/adding-a-comment-system/)
- [ ] 看板娘

<!-- more -->
## mkdocs教程

### install & run

安装mkdocs非常简单
```
pip install mkdocs-material
```

运行本地服务器
```
mkdocs serve -a 0.0.0.0:8000
```

生成静态页面，默认输出到`site/`目录
```
mkdocs build
```
### 配置

只有一个配置文件mkdocs.yml
```
mkdocs new .
```

```
.
├─ docs/
│  └─ index.md
└─ mkdocs.yml
```
## mkdocs blog支持

[Setting up a blog - Material for MkDocs (squidfunk.github.io)](https://squidfunk.github.io/mkdocs-material/setup/setting-up-a-blog/)
### tags索引页面

category和archive页面是自动生成的，tags页面需要手动配置
> mkdocs automatically generating [archive](https://squidfunk.github.io/mkdocs-material/plugins/blog/#archive) and [category](https://squidfunk.github.io/mkdocs-material/plugins/blog/#categories) indexes, [post slugs](https://squidfunk.github.io/mkdocs-material/plugins/blog/#config.post_url_format), configurable [pagination](https://squidfunk.github.io/mkdocs-material/plugins/blog/#pagination) and more.

想要创建一个页面包含所有tag索引，需要创建`docs/tags.md`，内容如下
```
# 标签

[TAGS]
```

mkdocs启用tags plugin
```yml
plugins:
  - tags:
      # enabled: !ENV [DEPLOY, false]
      enabled: true
      tags_file: tags.md
```
### 其它

#### excerpt

使用`<!-- more -->`分隔博客
*p.s 中间必须保留空格，使用`<!--more-->`无法生效*
#### authors

create the `.authors.yml` file in your blog directory
```yml
authors:
  squidfunk:
    name: Martin Donath
    description: Creator
    avatar: https://github.com/squidfunk.png
```

#### linking

> While [post URLs](https://squidfunk.github.io/mkdocs-material/plugins/blog/#config.post_url_format) are dynamically computed, the [built-in blog plugin](https://squidfunk.github.io/mkdocs-material/plugins/blog/) ensures that all links from and to posts and a post's assets are correct.

```
[Hello World!](blog/posts/hello-world.md)
```

链接到一个页面，同样只需要指向该页面的索引markdown
```
[Blog](../index.md)
```

#### readtime

自动启用，如果嫌估计的不准确，可以手动设置
```
---
date: 2023-01-31
readtime: 15
---
```
### 问题

#### 没有category页面

build后才会生成，直接serve是看不到的

### 有用的Plugin

#### meta

可以为一个目录的所有文件设置基础meta信息，在分门别类存储博客时比较有用。但是目前是需要付费才能使用
**Sponsors only** – this plugin is currently reserved to [our awesome sponsors](https://squidfunk.github.io/mkdocs-material/insiders/).

[Built-in meta plugin - Material for MkDocs (squidfunk.github.io)](https://squidfunk.github.io/mkdocs-material/plugins/meta/)
The meta plugin solves the problem of setting metadata (front matter) for all pages in a folder, i.e., a subsection of your project, which is particularly useful to ensure that a certain subset of pages features specific tags, uses a custom template, or is attributed to an author.


## Deploy

参考资料：
- mkdocs关于deploy文档：[Publishing your site - Material for MkDocs (squidfunk.github.io)](https://squidfunk.github.io/mkdocs-material/publishing-your-site/?h=deploy)
### cloudfare page

参考资料：
- mkdocs+cf page资料：[Deploy MkDocs with Material or Material Insiders theme to Cloudflare Pages - Starfall Projects](https://www.starfallprojects.co.uk/projects/deploy-host-docs/deploy-mkdocs-material-cloudflare/#site-setup)
- cf page文档：[Get started guide · Cloudflare Pages docs](https://developers.cloudflare.com/pages/get-started/guide/)
- cloudflare build platform介绍: [Modernizing the toolbox for Cloudflare Pages builds](https://blog.cloudflare.com/moderizing-cloudflare-pages-builds-toolbox/)

cf page支持很多preset（各种静态网站框架），mkdocs是其中之一，因此配置非常简单。

*注意：需要配置`requirements.txt`文件，否则cf build时会报错`mkdocs not found`*
### github page

github支持多个page！[关于 GitHub Pages - GitHub 文档](https://docs.github.com/zh/pages/getting-started-with-github-pages/about-github-pages)
> 有三种类型的 GitHub Pages 站点：项目、用户和组织。 项目站点连接到 GitHub 上托管的特定项目，例如 JavaScript 库或配方集合。 用户和组织站点连接到 GitHub.com 上的特定帐户。
> 若要发布用户站点，必须创建名为 `<username>.github.io` 的个人帐户拥有的存储库。 若要发布组织站点，必须创建名为 `<organization>.github.io` 的组织帐户拥有的存储库。 除非使用的是自定义域，否则用户和组织站点在 `http(s)://<username>.github.io` 或 `http(s)://<organization>.github.io` 中可用。
> 项目站点的源文件与其项目存储在同一个仓库中。 除非使用的是自定义域，否则项目站点在 `http(s)://<username>.github.io/<repository>` 或 `http(s)://<organization>.github.io/<repository>` 中可用。

## markdown格式修复

原本写的markdown文件有一些格式不太规范，切换成mkdocs后，有许多报错信息。以下列出其中一些问题，并且提供一个自动修复脚本[fix_markdown.py](../../code/fix_markdown.py)，避免一些枯燥的手动修改。
### metadata

markdown开头部分可以定义一些元数据，如作者，日期等，这部分叫做frontmatter。通常采用yaml格式。

日期只能使用`YY-mm-dd`或者`YY-mm-dd HH:MM:SS`，像`2023-04-20 16:22`这样的格式是无法识别的
```
ERROR   -  Error reading metadata 'date' of post 'blog/posts/2023-04-20-具体数学.md' in 'docs':
           Expected type: <class 'datetime.date'> or <class 'datetime.datetime'> but received: <class 'str'>
```
脚本支持修复该问题。

我使用obsidian的Template功能来自动生成metadata，其默认生成的时间格式是不带秒的时间格式，因而导致了以上问题。
可以自定义其模板时间格式：[Templates - Obsidian Help](https://help.obsidian.md/Plugins/Templates)
修正后模板如下：
```
---
title: {{title}}
date: {{date}} {{time:HH:mm:ss}}
tags:
- linux
categories:
- 折腾
---

<!-- more -->
```
### 图片链接

原本使用了两种格式`/images/xxx/xxx.png`, `images/xxx/xxx.png`的图片链接，导致报错
```
Doc file 'blog/posts/2023-04-20-具体数学.md' contains a relative link 'images/具体数学/image-20220607171500766.png', but the target
           'blog/posts/images/具体数学/image-20220607171500766.png' is not found among documentation files
```

mkdocs所有链接均使用相对路径，改为相对于md文件的路径即可。
*p.s 这里不得不提下obsidian的方便之处了。obsidian采用了尽可能匹配的原则，因此只要文件名相同，无论位于哪个目录下，均能正确匹配*

使用正则修改关键代码如下
```python
RE = fr'!\[(.*?)\]\(/{old_prefix}(.*?)\)'   # old_prefix = images/
m1 = re.findall(RE, content)
if m1:
  content = re.sub(RE, f'![\g<1>]({new_prefix}\g<2>)', content)
```
### 内部链接

[Setting up a blog - Material for MkDocs (squidfunk.github.io)](https://squidfunk.github.io/mkdocs-material/setup/setting-up-a-blog/?h=relative+link#linking-from-and-to-posts)

mkdocs使用相对路径，指向引用的md文件，并且也支持使用`#`定位到特定section。原本hexo的链接格式则复杂多了。
由于数目不多，手动修改即可。
```
[openwrt配置#ipv6](2022-02-13-openwrt配置.md#ipv6)
```

*p.s obsidan同样支持定位到另一个markdown文档，并且不用太在意路径*
### 数学公式

[Math - Material for MkDocs (squidfunk.github.io)](https://squidfunk.github.io/mkdocs-material/reference/math/)
有MathJax和KaTex两个选择
- **Speed**: KaTeX is generally faster than MathJax. If your site requires rendering large quantities of complex equations quickly, KaTeX may be the better choice.
- **Syntax Support**: MathJax supports a wider array of LaTeX commands and can process a variety of mathematical markup languages (like AsciiMath and MathML). If you need advanced LaTeX features, MathJax may be more suitable.

但是使用两种方案，据无法正确显示我的[具体数学](2023-04-20-具体数学.md)那篇文章的所有数学公式，并且MathJax正确识别的更少。因此最后选用了KaTeX