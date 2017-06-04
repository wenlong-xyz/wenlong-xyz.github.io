---
title: Something about Hexo（持续更新）
date: 2016-05-15 17:04:19
category: Technology
tags: [Hexo]
---
记录Hexo 使用的使用技巧，并会持续更新
## 常用命令
```bash
hexo new [layout] <title>  #新建一篇文章
hexo generate              #生成静态文章
hexo generate -d           #文件生成后立即部署
hexo server                #启动服务器
hexo server -p [port]      #更改服务端口
hexo deploy                #部署网站

hexo new draft <filename> #新建草稿 
hexo server --drafts       #预览草稿
hexo publish [layout] <filename>  #发布草稿为文章

```
## 主题安装
```
git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant
npm install hexo-renderer-jade --save
npm install hexo-renderer-sass --save
```
## Maupassant配置修改
```
theme/maupassant/source/css/style.scss  #显示的css配置
```

## Problems
**Problem-1**
```
TypeError: Cannot set property 'lastIndex' of undefined
    at highlight (E:\workspaces\Dropbox\Hexo\node_modules\highlight.js\lib\highlight.js:471:35)
    at E:\workspaces\Dropbox\Hexo\node_modules\highlight.js\lib\highlight.js:524:21
    at Array.forEach (native)
    at Object.highlightAuto (E:\workspaces\Dropbox\Hexo\node_modules\highlight.js\lib\highlight.js:520:20)
    at highlight (E:\workspaces\Dropbox\Hexo\node_modules\hexo-util\lib\highlight.js:98:19)
    at highlightUtil (E:\workspaces\Dropbox\Hexo\node_modules\hexo-util\lib\highlight.js:21:14)
    at E:\workspaces\Dropbox\Hexo\node_modules\hexo\lib\plugins\filter\before_post_render\backtick_code_block.js:49:15
    at String.replace (native)
    at Hexo.backtickCodeBlock (E:\workspaces\Dropbox\Hexo\node_modules\hexo\lib\plugins\filter\before_post_render\backti
ck_code_block.js:15:31)
    at Hexo.tryCatcher (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\util.js:16:23)
    at Hexo.<anonymous> (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\method.js:15:34)
    at E:\workspaces\Dropbox\Hexo\node_modules\hexo\lib\extend\filter.js:68:35
    at tryCatcher (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\util.js:16:23)
    at Object.gotValue (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\reduce.js:145:18)
    at Object.gotAccum (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\reduce.js:134:25)
    at Object.tryCatcher (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\util.js:16:23)
    at Promise._settlePromiseFromHandler (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\promise.js:503:31)
    at Promise._settlePromise (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\promise.js:560:18)
    at Promise._settlePromiseCtx (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\promise.js:597:10)
    at Async._drainQueue (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\async.js:131:12)
    at Async._drainQueues (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\async.js:136:10)
    at Immediate.Async.drainQueues [as _onImmediate] (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\async.
js:16:14)
    at processImmediate [as _immediateCallback] (timers.js:383:17)
FATAL Cannot set property 'lastIndex' of undefined
TypeError: Cannot set property 'lastIndex' of undefined
    at highlight (E:\workspaces\Dropbox\Hexo\node_modules\highlight.js\lib\highlight.js:471:35)
    at E:\workspaces\Dropbox\Hexo\node_modules\highlight.js\lib\highlight.js:524:21
    at Array.forEach (native)
    at Object.highlightAuto (E:\workspaces\Dropbox\Hexo\node_modules\highlight.js\lib\highlight.js:520:20)
    at highlight (E:\workspaces\Dropbox\Hexo\node_modules\hexo-util\lib\highlight.js:98:19)
    at highlightUtil (E:\workspaces\Dropbox\Hexo\node_modules\hexo-util\lib\highlight.js:21:14)
    at E:\workspaces\Dropbox\Hexo\node_modules\hexo\lib\plugins\filter\before_post_render\backtick_code_block.js:49:15
    at String.replace (native)
    at Hexo.backtickCodeBlock (E:\workspaces\Dropbox\Hexo\node_modules\hexo\lib\plugins\filter\before_post_render\backti
ck_code_block.js:15:31)
    at Hexo.tryCatcher (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\util.js:16:23)
    at Hexo.<anonymous> (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\method.js:15:34)
    at E:\workspaces\Dropbox\Hexo\node_modules\hexo\lib\extend\filter.js:68:35
    at tryCatcher (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\util.js:16:23)
    at Object.gotValue (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\reduce.js:145:18)
    at Object.gotAccum (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\reduce.js:134:25)
    at Object.tryCatcher (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\util.js:16:23)
    at Promise._settlePromiseFromHandler (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\promise.js:503:31)
    at Promise._settlePromise (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\promise.js:560:18)
    at Promise._settlePromiseCtx (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\promise.js:597:10)
    at Async._drainQueue (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\async.js:131:12)
    at Async._drainQueues (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\async.js:136:10)
    at Immediate.Async.drainQueues [as _onImmediate] (E:\workspaces\Dropbox\Hexo\node_modules\bluebird\js\release\async.
js:16:14)
    at processImmediate [as _immediateCallback] (timers.js:383:17)
```
**Solution-1**
```
set auto_detect(in _config.yml) to false solved my problem.
```
