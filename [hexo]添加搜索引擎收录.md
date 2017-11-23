默认状态下`huberyhe.github.io`没有被搜索引擎收录，谷歌和百度并不知道这个网站的存在，这需要我们分别到谷歌和百度的站长工具里提交我们的网址，为了让他们确信我们是这个网址的所有者，我们需要按他们的要求在网站用添加特殊的标记。
常见的有文件验证和html标签验证，对于hexo搭建的网站，使用html标签更加直白。
html标签验证需要我们在首页html的head部添加类似以下格式的标签，其中content的内容是域唯一的。
```html
<meta name="google-site-verification" content="n3vW5VWa0dZmPY-AA8UAp01xuVPkN5_IiepW50blPDA">
<meta name="baidu-site-verification" content="bNiXz6eKG6">
```
_ _ _
以下步骤是将这类标签更加合理地添加到hexo博客中的一种方式。
### 1.在hexo的配置文件中添加必须的字段
```bash
cd hexo_blog
vim _config.yml
```
添加：
```yaml
# Site Varifycation
site_verification:
    google: ABCDEFG_XYZ
    baidu: ABCDEFG_XYZ
```
### 2.编辑主题
```bash
vim themes/oishi/layout/_partial/head.ejs
```
在`<head>`和`</head>`之间添加
```html
<% if (config.site_verification.google){ %>
  <meta name="google-site-verification" content="<%= config.site_verification.google %>" />
<% } %>
<% if (config.site_verification.baidu){ %>
  <meta name="baidu-site-verification" content="<%= config.site_verification.baidu %>" />
<% } %>
```
### 3.验证
提交上传之后就可以通过验证，并且很快就可以在搜索引擎中搜索到`huberyhe.github.io`（百度要慢一些）