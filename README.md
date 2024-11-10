# ChatNio Deploy

最近一则新闻 [15 岁山东初中生做 CTO，开源项目刚刚被数百万元收购了](https://36kr.com/p/3021812338042369) 火爆出圈，笔者看了下被收购的开源项目 [ChatNio](https://github.com/zmh-program/chatnio)，发现挺好用了，就体验下。

ChatNio 号称**下一代 AIGC 一站式商业解决方案**，其功能融合了 Next Web 和 One API 各自的优点，通过整合实现 1+1>2 的效果：

> Chat Nio > [Next Web](https://github.com/ChatGPTNextWeb/ChatGPT-Next-Web) + [One API](https://github.com/songquanpeng/one-api)

ChatNio 在保持界面简洁、美观的同时，支持丰富的功能。

![](https://raw.githubusercontent.com/zmh-program/chatnio/refs/heads/main/screenshot/chatnio.png)

面向所有用户提供通用的基本功能：

1. 支持多种模型
2. 支持本地模型，如 Ollama
3. 支持分布式流式传输，图像生成
4. 对话跨设备自动同步和分享
5. 预设市场，即预定义提示词
6. AI 批量文章生成
7. AI 项目生成器

面向`企业级用户`的功能：

1. 齐全的后台管理
2. 实现订阅和 Token 弹性计费系统
3. 优秀渠道管理
4. Key 中转服务
5. 多模型聚合支持
6. 全部模型都支持联网搜索
7. 多种兑换码体系
8. 商用友好的开源协议

更详细的特性介绍见 [代码仓库-README.features](https://github.com/zmh-program/chatnio?tab=readme-ov-file#-features)。

## 部署

由于 ChatNio 吸收了 One API，Next Web 等优秀特性，使得 ChatNio 的部署要比 LobeChat、Dify 等简单得多。只需要 MySQL、Redis 和 ChatNio 3 个容器，就能运行起来。

1. clone 代码仓库

```shell
git clone https://github.com/dockerq/chatnio-deploy.git
```

2. 下载镜像

```shell
# 进入 clone 的代码仓库目录
cd chatnio-deploy
docker compose pull
[+] Pulling 3/3
 ✔ mysql Pulled                                                                                                       4.5s
 ✔ chatnio Pulled                                                                                                     4.7s
 ✔ redis Pulled                                                                                                       4.4s
```

3. 启动容器

```shell
docker compose up -d
[+] Running 3/0
 ✔ Container redis    Running                                                                                         0.0s
 ✔ Container db       Running                                                                                         0.0s
 ✔ Container chatnio  Running                                                                                         0.0s
```

从步骤 3 的输出可以看到，一共运行了 3 个容器：redis、db(MySQL) 和 chatnio，其中 chatnio 容器包含了前端和后端，后端通过静态文件的方式输出前端页面。

因为这里服务是部署在自己电脑上的，所以在浏览器输入`localhost:8094`即可进入 ChatNio 的前端页面：

![](https://github.com/alwqx/picx-images-hosting/raw/master/blog/2024/chatnio-home.b8x4cvsfi.webp)

可以看到，前端页面非常的`简洁`、`美观`，比较符合作者的审美，也提升了`易用性`。

根据 [ChatNio Docs](https://www.chatnio.com/docs/deploy#%E4%BF%AE%E6%94%B9%E5%AF%86%E7%A0%81) 的描述：

> 管理员账号为 `root`，密码默认为 `chatnio123456`

登录成功后会提示**修改管理员密码**。

## 配置

**只有管理员权限的帐号，才能进入后台页面进行配置**。

### Ollama

ChatNio 除了支持各大模型厂商的 API 外，还支持本地 AI，比如 Ollama。

> 不过并不像 LobeChat 那样在`模型供应商`配置中提供明确的 Ollama 选项，而是**统一放在了 OpenAI 这个选项中**。
> 也就是说，任何兼容 OpenAI API 的本地接口，都能接入到 ChatNio 的模型供应商中。

接入 Ollama 需要在 ChatNio 管理后台做 2 步操作：`创建渠道`和`模型市场`。

首先，我们创建 Ollama 的渠道：管理员帐号登录，进入`后台管理`，点击`渠道设置` -> `创建渠道`：

![](https://github.com/alwqx/picx-images-hosting/raw/master/blog/2024/chatnio-config-ollama.5tr1kimndc.webp)

注意，**这里填的模型，要和 ollama list 中的模型 Name 一致**。

```shell
$ ollama list
NAME                       ID              SIZE      MODIFIED
qwen2:7b                   e0d4e1163c58    4.4 GB    5 months ago
```

其次，点击`模型市场`，页面会显示刚才创建的渠道模型：

![](https://github.com/alwqx/picx-images-hosting/raw/master/blog/2024/chatnio-config-model.7sn8auxk49.webp)

点击该渠道模型，进入预设好的该渠道的模型新建表单：

![](https://github.com/alwqx/picx-images-hosting/raw/master/blog/2024/chatnio-config-model-create.ic4zt937w.webp)

这里更新下模型图片和标签后，点击`提交`即可。

这时候回到对话页面，在页面下方选择新添加的模型，就能够和模型对话啦：

![](https://github.com/alwqx/picx-images-hosting/raw/master/blog/2024/chatnio-model-chat.26lhx01rwn.webp)

### 联网搜索 SearXNG

ChatNio 默认使用 `duckduckgo-api` 进行网页搜索，但是当你访问该 API 地址 `https://duckduckgo-api.vercel.app`时，你会发现不能使用：

![](https://github.com/alwqx/picx-images-hosting/raw/master/blog/2024/chatnio-network-duck.39l77w411r.webp)

可能因为使用太多，API 提供方做了限制吧。ChatNio 推荐使用 [SearXNG](https://github.com/searxng/searxng) 作为搜索 API，这里笔者也推荐 SearXNG，它不仅提供联网搜索 API，还能作为搜索引擎单独使用，保护个人隐私。

首先，参考之前的文章 [大模型联网搜索组件 SearXNG 部署和使用](https://mp.weixin.qq.com/s/AQWdjxBPuYOFqG1vItuTqQ) 搭建好 SearXNG 服务。

然后在配置中填写 SearXNG 的 API，并保存。

![](https://github.com/alwqx/picx-images-hosting/raw/master/blog/2024/chatnio-network-searxng.3d4t5m3p9l.webp)

**回到对话页面，开启联网搜索**，查看下使用效果：

![](https://github.com/alwqx/picx-images-hosting/raw/master/blog/2024/chatnio-network-demo.1e8mfaddfp.webp)

![](https://github.com/alwqx/picx-images-hosting/raw/master/blog/2024/chatnio-network-genius.5j47regth6.webp)

> 联网搜索的原理，就是将用户输入的消息先联网查询，然后将查询的结果聚合到用户的消息中，一起发给模型，由模型回复信息。

用户可以随时开启或者关闭联网搜索功能。

## 总结

通过使用和部署，ChatNio 具有以下特点：

1. UI 界面简洁、美观、易用
2. 部署简单易用，基本上开箱即用，同级别的软件 LobeChat 比较难部署
3. 用户管理功能
4. 平台化功能强，**集成了很多企业级的功能**，包括`仪表盘`、`渠道管理`、`计费`、`订阅`、`负载均衡`等丰富的功能
5. 原生支持`联网搜索`
6. 图片对话支持较差，这点不如 LobeChat

![](https://github.com/alwqx/picx-images-hosting/raw/master/blog/2024/chatnio-dashboard.58hdyaaqd6.webp)

综上，ChatNio 是一款优秀的 AIGC 前端软件，相比于同类型开源软件，它有很多差异性且优秀的功能。因此它具有广泛的使用场景，无论是个人使用、站长使用、小公司内部使用或者二次开发，都非常便捷。

对于只有对话和生成图片等基本需求的用户，个人是很推荐使用 ChatNio 的。

另外，由于时间等因素限制，本文并没有评测 ChatNio 的文件上传功能，因此并不确定 ChatNio 是否具备**知识库**的能力。这方便可以后续深入使用后再介绍了。
