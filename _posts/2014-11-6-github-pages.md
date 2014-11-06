---
layout: post
title: How I get my GitHub Pages to work
---

###GitHub Pages有两种建站方法
- user websites，如[http://wuhui519.github.io](http://wuhui519.github.io)。 user websites不需要分支，直接在master上修改提交，然后jeykll会自动解析master分支生成静态网页
- project websites，如[http://wuhui519.github.io/testblog/](http://wuhui519.github.io/testblog/)。 project websites原本是为github上各种开源项目设置的静态网页，用于介绍开源项目的各方面情况，但是实际上也可以用于个人博客，project websites必须在github项目的gh-pages分支上提交，只有在这个分支下的文件才会被GitHub Pages解析

###了解了这两种建站方式的区别，就可以选择建站的模板了，我的两个pages选择了两种不同的模板
- [https://github.com/barryclark/jekyll-now](https://github.com/barryclark/jekyll-now)
- [https://github.com/minixalpha/StrayBirds](https://github.com/minixalpha/StrayBirds)

###使用模板建站的步骤非常简单，以jekyll-now为例：
1. Fork jeykyll-now到本人GitHub帐号
2. 点击分支的Settings按钮，修改repository的名字为wuhui519.github.io，此时刷新wuhui519.github.io就可以看到正常运行的网页了
3. 修改_config.yml里面各种帐号详情，如名字、介绍、联系方式等
4. 所有的文章都放在/_posts/文件夹下，jeykll支持markdown文件的解析，新建的markdown文件以year-month-day-title.md的格式命名。重要的一点是，markdown文件必须包含[Additional front-matter variables](http://jekyllrb.com/docs/frontmatter/)，格式为：
\---\n
layout: post\n
title: Blogging Like a Hacker\n
\---\n
front-matter必须放在第一行，前面不能有空格或换行，否则会造成build错误
5. commit新建的md文件，再次刷新wuhui519.github.io文件，就可以看到新增的文章了

###如果文件提交后jeykll解析错误，会收到一封GitHub Page build failure邮件。根据GitHub的提示，build失败的常见原因在[这里](https://help.github.com/articles/troubleshooting-github-pages-build-failures)。我第一次碰到这个错误时，根据issue里别人的提示，重新fork一次就好了。其它时候一般是front-matter格式错误。

###解决了这些问题后，就可以使用GitHub Pages发文章了。当然，使用标签、归档、评论等功能需要使用jeykll的插件，但是GitHub Pages支持的jeykll插件不多。要使用插件的话，一般使用jeykll本地build后，直接上传build后的静态网页可以解决这个问题。Jeykll更多的高级特性继续发掘中。。。

####参考
- [搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)
- [Build A Blog With Jekyll And GitHub Pages](http://www.smashingmagazine.com/2014/08/01/build-blog-jekyll-github-pages/)
