博客最重要的作用除了记录之外，还有分享和交流。而一个没有评论功能的博客，又谈何分享，谈何交流。  

正因为评论系统对博客的重要性，我搭建了基于 [Gitment](https://github.com/imsun/gitment) 的评论系统，而这篇博文将记录这个过程。

> Gitment is a comment system based on GitHub Issues, which can be used in the frontend without any server-side implementation.

# 选择评论系统
* 网易云跟帖、多说、畅言、友言：这些国内第三方评论系统，由于一些有关部门的原因（纯属个人臆测），都已经挂了，也是连体验的机会都没有
* disqus、facebook：这两个需要科学上网，门槛较高，国内用户使用便利性一般
* [discourse](https://www.discourse.org/)：收费，standard 版要100刀／月，暂不考虑
* [来必力](https://livere.com/)：这款中文名字怪异的评论系统出自高丽棒子之手，支持中文、英文、韩文，支持多种社交账号评论，功能齐全，致命硬伤是服务器太慢导致用户体验差（经常是直接加载不出来，体验的机会的没有）
* [Gitment](https://github.com/imsun/gitment): 基于 GitHub Issues 的评论系统，需要登录 GitHub 才能评论，但考虑到博客受众，这个可以接受

# 初次尝试
开始时，我尝试用来必力搭建评论系统，但来必力的服务器太慢（估计没有国内服务器），评论功能经常加载不出、无法使用，只好放弃。

# 改用 Gitment
用 Gitment 搭建评论系统较为简单，分四步完成：
## 1、引入 css 和 js
```html
<link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css">
<script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
```
也可以通过 npm 引入（我的是 ruby 环境，不采用 npm 引入）
```shell
$ npm i --save gitment
```
```js
import 'gitment/style/default.css'
import Gitment from 'gitment'
```
## 2、注册 OAuth Application
[点击这里](https://github.com/settings/applications/new)在 GitHub 注册一个 OAuth application. 注册完后记下 __client ID__ 和 __client secret__


__注意：请正确填写 Callbak URL，一般来说填写首页 URL 即可__

## 3、用 js 渲染组件
```js
const gitment = new Gitment({
  id: 'Your page ID', // optional
  owner: 'Your GitHub ID',
  repo: 'The repo to store comments',
  oauth: {
    client_id: 'Your client ID',
    client_secret: 'Your client secret',
  },
  // ...
})

gitment.render('comments')
// or
// gitment.render(document.getElementById('comments'))
// or
// document.body.appendChild(gitment.render())
```

## 4、初始化评论
最后，到每篇博客里，点击 __Initialize__ 按钮，初始化对应博文的评论（实际上是创建博文对应的 GitHub Issue）。

# Gitment 问题
搭建完初始化评论时，Gitment报错。  
* **错误信息：**Validation error
* **HTTP 状态码**：422

## 原因

Gitment 初始化评论时，会发送 `https://api.github.com/repos/yeehomfoo/yeehomfoo.github.io/issues` 请求，博文链接放在请求参数的 labels 字段里。而这个博文链接包含文章中文标题，中文会被 encode，encode 之后的长度超过了 GitHub 最大 label 长度（50）限制。
```json
// 请求参数
{
  "title":"在 Mac中搭建 Node.js环境 - Wild of Knowledge",
  "labels":["gitment",
            "https://yeehomfoo.github.io/posts/%25E5%259C%25A8--mac%E4%B8%AD%E6%90%AD%E5%BB%BA-node.js%E7%8E%AF%E5%A2%83/"
            ],
  "body":"https://yeehomfoo.github.io/posts/%E5%9C%A8-mac%E4%B8%AD%E6%90%AD%E5%BB%BA-node.js%E7%8E%AF%E5%A2%83/\n\n"
}

```

## 解决方案

查看 Gitment 源码发现，Gitment 直接使用博文链接作 labels 参数，不做任何处理
```js
{
    key: 'createIssue',
    value: function createIssue() {
      var _this5 = this;

      var id = this.id,
          owner = this.owner,
          repo = this.repo,
          title = this.title,
          link = this.link,
          desc = this.desc,
          labels = this.labels;

      return _utils.http.post('/repos/' + owner + '/' + repo + '/issues', {
        title: title,
        labels: labels.concat(['gitment', id]),
        body: link + '\n\n' + desc
      }).then(function (meta) {
        _this5.state.meta = meta;
        return meta;
      });
    }
}
```

修改源码，当博文链接超长时，增加处理逻辑
```js
{
    key: 'createIssue',
    value: function createIssue() {
      var _this5 = this;

      var id = encodeURI(this.id),
          owner = this.owner,
          repo = this.repo,
          title = this.title,
          link = this.link,
          desc = this.desc,
          labels = this.labels;

      if (id && id.length > 50) {
        id = id.substr(0, 50);
      }

      return _utils.http.post('/repos/' + owner + '/' + repo + '/issues', {
        title: title,
        labels: labels.concat(['gitment', id]),
        body: link + '\n\n' + desc
      }).then(function (meta) {
        _this5.state.meta = meta;
        return meta;
      });
    }
}
```
在获取评论时，也要通过博文链接这个 label 来获取，所以获取评论源码也需要修改。最初我打算修改成通过 title 查询，而查看[官方文档](https://developer.github.com/v3/issues/#list-issues-for-a-repository)发现相关接口不支持通过 title 查询。最终还得增加处理博文链接超长情况的逻辑。
```js
{
    key: 'loadMeta',
    value: function loadMeta() {
      var _this7 = this;

      var id = encodeURI(this.id),
          owner = this.owner,
          repo = this.repo;

      if (id && id.length > 50) {
        id = id.substr(0, 50);
      }

      return _utils.http.get('/repos/' + owner + '/' + repo + '/issues', {
        creator: owner,
        labels: id
      }).then(function (issues) {
        if (!issues.length) return Promise.reject(_constants.NOT_INITIALIZED_ERROR);
        _this7.state.meta = issues[0];
        return issues[0];
      });
    }
}
```


# 总结
至此，一个博客评论系统搭建完成了。  
搭建评论系统和写这篇博文差不多耗费了我一天的时间，填坑速度亟待提高啊！


(done)


## 参考文档

[1] [Gitment](https://github.com/imsun/gitment)


[2] [GitHub REST API v3](https://developer.github.com/v3/)
