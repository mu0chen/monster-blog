
---
title: something
date: 2022-04-25
tags: 
- user
- mmm
categories:
- 工具类
---

我们都知道hexo作为一个用node.js打造的博客框架，能够很好的适配github，因此也能够很方便的将我们的博客同步到Github Page中去，但吹毛求疵的讲，这样还是有一些不利的因素在里面：

1. 本地写好博客后，每次提交都需要使用Git命令进行提交
1. 当更换设备后，无法很好的同步到Github Page中，若需要同步，首先需要Pull仓库，然后再进行修改，国内的Github链接懂得都懂。<!-- more -->

因此，就有了如下的需求：

1. 希望能够在任何设备上都能够很方便的完成博客内容的更新
1. 提交方式更为便捷

故而，本次采用的是**语雀+Github Page+Hexo+腾讯云函数**来完成个人主页的搭建。
语雀能够很好的保证我们在不同设备之间的切换，并且可以完美适配Markdown语法，因此，这样做的性价比我认为是最高的。

# 创建Github Page

1. 登录自己的Github
1. 创建公开项目，项目名字：username.github.io
# Hexo配置和设置
首先最基本的安装Hexo并配置相应的主题：
```shell
npm install -g hexo-cli
hexo init myBlog

```
# 创建Github源码仓库
要注意的是我们需要**将Hexo中的源码push到该仓库中**，注意该仓库与Github Pages不是一个仓库。我们需要通过这个Github的Hexo中的源码仓库，将最终生成的页面发布到我们的Github Pages中，因此，需要配置该源码仓库的Github Actions
```yaml
# workflow name
name: Deploy To Github Pages

# 当有 push 到仓库和外部触发的时候就运行
on: [push, repository_dispatch]

# YUQUE_TOKEN
# Github_SSH_PRIVATE_KEY

jobs:
  deploy: 
    name: Deploy Hexo Public To Pages
    runs-on: ubuntu-latest 
    env:
      TZ: Asia/Shanghai    
        
    steps:
    # check it to your workflow can access it
    # from: https://github.com/actions/checkout
    - name: Checkout Repository master branch
      uses: actions/checkout@master 
      
    # from: https://github.com/actions/setup-node  
    - name: Setup Node.js 10.x 
      uses: actions/setup-node@master
      with:
        node-version: "10.x"
    
    # from https://github.com/x-cold/yuque-hexo
    - name: Setup Hexo Dependencies and Generate Public Files
      env:
        YUQUE_TOKEN: ${{ secrets.YUQUE_TOKEN }}
      run: |
        npm install hexo-cli@1.11.0 -g
        npm install yuque-hexo -g
        npm install
        npm run start
        
    # from https://github.com/peaceiris/actions-gh-pages    
    - name: Deploy
      uses: peaceiris/actions-gh-pages@v3
      with:
          deploy_key: ${{ secrets.HEXO_DEPLOY_PRIVATE_KEY}}
          external_repository: username/username.github.io
          publish_branch: master
          publish_dir: ./public
          commit_message: ${{ github.event.head_commit.message }}
```
注意，此时的Github Actions还不能够运行，因为在上述中有两个重要的参数是没有配置的，一个是YUQUE_TOKEN，一个是HEXO_DEPLOY_PRIVATE_KEY

1. YUQUE_TOKEN——是语雀官方给出的，我们想要通过第三方（yuque-hexo）去访问语雀里面的内容就需要这个token值，获取方式为语雀上点击个人头像 –> 设置 –> Token 即可获取，并为了能够让此workflow能够正常运行，就需要在该仓库中的 Settings–>Secrets 中进行添加，对重要信息进行保密。
1. HEXO_DEPLOY_PRIVATE_KEY——这个就是是SSH-key密钥中的私钥，同样需要在Secrets中进行添加，公钥（.pub）已经存储在 Github 中
# 配置腾讯云函数

1. 登录腾讯云，搜索云函数，创建
1. 选择python，2.7 和 3.6 都行，空白函数
1. 运行角色，SCF_QcsRole即可
1. 注意执行方法，有强制要求

直接使用2.7，配置：
```python
import requests 
    def main_handler(event, context): 
        r = requests.post("https://api.github.com/repos/用户名/仓库名/dispatches", 
                          json = {"event_type": "start"}, 
                          headers = {"User-Agent":'curl/7.52.1', 
                                     'Content-Type': 'application/json', 
                                     'Accept': 'application/vnd.github.everest-preview+json', 
                                     'Authorization': 'token Github访问Token'}) 
        if r.status_code == 204: 
            return "This's OK!" 
        else: 
            return r.status_code
```
可以测试一下看看Github Action有没有正常触发

# 配置语雀
## 语雀部分

1. 注册，登录
1. 创建知识库–>文档知识库–>可见范围为互联网可见
1. 工作台–>知识库–>找到新创建的知识库，管理–>设置–>路径进行自定义
1. 工作台–>知识库–>找到新创建的知识库，管理–>设置–>开发者–>名称任意。URL 为云函数的地址，即上面获取到的访问路径
## Hexo源码部分
简单来说，我们需要做的就是要配置好，当云函数那边的请求发送过来，触发了Github Actions时，Github Actions在运行到：
```shell
npm install hexo-cli -g
npm install yuque-hexo -g
npm install
npm run start
```
即为最后一部分run start时，要能够完成从hexo源码（一些markdown文件）==>真正的HTML网页，因此这就需要借助于上述中提到的yuque-hexo工具，它主要是完成一个脚本命令，将编译、发布等命令合并成一个，因此，我们可以在本地配置，也可以直接在Github源码中配置相应的文件，这里直接在Github源码中配置相应的脚本文件，package.json中增加以下部分：

```json
{
  "scripts": {
    "start": "yuque-hexo clean && yuque-hexo sync && hexo clean && hexo generate",
    "clean": "hexo clean",
    "build": "hexo generate",
    "deploy": "yuque-hexo sync && hexo deploy",
    "server": "hexo server",
    "clean:yuque": "yuque-hexo clean",
    "publish": "npm run clean && npm run deploy",
    "dev": "hexo s",
    "sync": "yuque-hexo sync",
    "reset": "npm run clean:yuque && npm run sync"
  },
  "yuqueConfig": {
    "baseUrl": "https://www.yuque.com/api/v2",
    "login": "语雀的用户名",
    "repo": "语雀的知识库名称",
    "mdNameFormat": "title",
    "postPath": "source/_posts",
    "onlyPublished": false
  }
}
```
# ERROR
在这个过程中Github Actions可能会发生以下错误：
```shell
FATAL { err:
   TypeError: line.matchAll is not a function...
```
这个主要是因为我们在Github Actions中的脚本执行过程是这样的：
```shell
npm install hexo-cli -g
npm install yuque-hexo -g
npm install
npm run start
```
与本地不同的时，这是没有指定hexo-cli和yuque-hexo版本的，因此执行相应的命令时，hexo-cli和yuque-hexo总会被采用最新的，而Actions中的Node.js版本是10.x版本的，最新的hexo-cli需要12.x的Node.js，因此更改一下Action脚本中的Node.js版本即可，或者可以通过指定hexo-cli和yuque-hexo版本的方式，同样也可以达到解决该问题的效果。

# 主要参考
[https://www.zhwei.cn/hexo-github-actions-yuque/](https://www.zhwei.cn/hexo-github-actions-yuque/)
