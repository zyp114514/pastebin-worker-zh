# Pastebin-worker
翻译自：[SharzyL的项目](https://github.com/SharzyL/pastebin-worker)

介绍：
这是一个可以部署在 Cloudflare workers 上的 pastebin（代码片段分享网站）。你可以在 [shz.al](https://shz.al) 上试用它。

**理念**：轻松部署，友好的命令行使用，丰富的功能。

**特点**：

1. 用短至 4 个字符的链接分享你的代码片段
2. 自定义你的代码片段链接
3. 随心所欲地**更新**和**删除**你的代码片段
4. 让你的代码片段在一段时间后**过期**
5. 由 PrismJS 提供的**语法高亮**
6. 将**markdown**文件显示为 HTML
7. 用作 URL 缩短器
8. 自定义返回的 mimetype

## 使用方法

1. 你可以直接在网站上（比如 [shz.al](https://shz.al)）发布，更新，删除你的代码片段。

2. 它还提供了一个方便的 HTTP API 来使用。详见 [API 参考](doc/api.md)。你可以通过命令行（使用 `curl` 或类似的工具）轻松地调用 API。

3. [pb](/scripts) 是一个 bash 脚本，可以让你在命令行上更容易地使用它。

## 限制

1. 如果部署在 Cloudflare Worker 免费套餐上，该服务每天最多允许 100,000 次读取，1000 次写入，1000 次删除。
2. 由于 Cloudflare KV 存储的大小限制，每个代码片段的大小限制在 25 MB 以下。

## 部署

如果你在 Cloudflare 上托管了你的域名，你可以自由地在你自己的域名上部署这个 pastebin。

需求：
1. \*nix 环境，有 bash 和基本的命令行程序。如果你使用 Windows，可以尝试 cygwin，WSL 或其他类似的工具。
2. GNU make.
3. `node` 和 `yarn`.

在 Cloudflare workers 仪表盘上创建两个 KV 命名空间（一个用于生产，一个用于测试）。记住它们的 ID。如果你不需要测试，只需创建一个。

克隆仓库并进入目录。使用 `wrangler login` 登录到你的 Cloudflare 账户。根据你自己的账户信息修改 `wrangler.toml` 中的条目（你需要修改的是 `account_id`，路由，kv 命名空间 id）。如果你不需要测试部署，可以安全地删除 `env.preview` 部分。参考 [Cloudflare 文档](https://developers.cloudflare.com/workers/cli-wrangler/configuration) 来找出这些参数。

修改 `config.json` 中的内容（它控制了静态页面的生成）：`BASE_URL` 是你的网站的 URL（没有结尾的斜杠）；`FAVICON` 是你想在你的网站上使用的 favicon 的 URL。如果你需要测试，也要修改 `config.preview.json`。

部署并享受吧！

```shell
$ yarn install
$ make deploy
```

## 认证

如果你想要一个私有的部署（只有你可以上传代码片段，但是每个人都可以阅读代码片段），在你的 `config.json` 中添加以下条目（其他配置也包含在最外层的括号中）：

```json
{
  "basicAuth": {
    "enabled": true,
    "passwd": {
      "admin1": "this-is-passwd-1",
      "admin2": "this-is-passwd-2"
    }
  }
}
```

现在，每次访问 PUT 或 POST 请求，以及每次访问首页，都需要一个 HTTP 基本认证，使用上面列出的用户-密码对。例如：

```shell
$ curl example-pb.com
需要 HTTP 基本认证

$ curl -Fc=@/path/to/file example-pb.com
需要 HTTP 基本认证

$ curl -u admin1:wrong-passwd -Fc=@/path/to/file example-pb.com
错误 401：HTTP 基本认证的密码不正确

$ curl -u admin1:this-is-passwd-1 -Fc=@/path/to/file example-pb.com
{
  "url": "https://example-pb.com/YCDX",
  "suggestUrl": null,
  "admin": "https://example-pb.com/YCDX:Sij23HwbMjeZwKznY3K5trG8",
  "isPrivate": false
}
```
