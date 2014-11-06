---
layout: post
title: How I get my GitHub Pages to work
---

###GitHub Pages有两种建站方法
- user websites，如http://wuhui519.github.io。 user websites不需要分支，直接在master上修改提交，然后jeykll会自动解析master分支生成静态网页
- project websites，如http://wuhui519.github.io/testblog/。 project websites原本是为github上各种开源项目设置的静态网页，用于介绍开源项目的各种情况，但是实际上也可以用于个人博客，project websites必须在github项目的gh-pages分支上提交，只有在这个分支下的文件才会被GitHub Pages解析

###了解了这两种建站方式的区别，就可以选择建站的模板了，我的两个pages选择的模块是两种模板
- https://github.com/barryclark/jekyll-now
- https://github.com/minixalpha/StrayBirds

###使用模板建站的步骤非常简单，以jekyll-now为例：
1. Fork jeykyll-now到本帐号github
2. 点击分支的Settings按钮，修改repository的名字为wuhui519.github.io，此时刷新wuhui519.github.io就可以看到正常运行的网页了
3. 修改_config.yml里面各种帐号详情，如名字、介绍、联系方式等
4. 所有的文章都放在/_posts/文件夹下，jeykll支持markdown文件的解析，新建的markdown文件以year-month-day-title.md的格式命名。重要的一点是，markdown文件必须包含[Additional front-matter variables](http://jekyllrb.com/docs/frontmatter/)，格式为：
\---
layout: post
title: Blogging Like a Hacker
\---
front-matter必须放在第一行，前面不能有空格或换行，否则会造成build错误
5. commit新建的md文件，再次刷新wuhui519.github.io文件，就可以看到新增的文章了


