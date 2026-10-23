之前的改 BUG 流程：

1、开发人工去飞书上复制“缺陷详细描述”
2、开发人工把“缺陷详细描述” + “修复期望”发送给 Codex
3、Codex 修复 bug
4、开发人工提交 git fix 记录
5、开发人工去飞书上更新 bug 状态为 Resolved
6、开发人工去飞书上给 bug 评论“已修复，见 v3.0.0”之类的内容

期望的改 BUG 流程：

1、开发人工在 Codex 里输入“修复 bug：{飞书的 bug link}”

2、Codex 自动：

* 读取飞书项目缺陷，包括“缺陷名称” + “缺陷详细描述”

* 向开发收集“修复期望”

* 综合前三个信息来修复 bug

* 运行测试 + 征求人工验证

* 提交 Commit + Push

* 修改飞书 bug 状态

* 添加飞书“已修复，见 v3.0.0”之类的评论

## 一、安装飞书官方 [Meegle CLI](https://github.com/larksuite/meegle-cli/blob/main/README.zh-CN.md)（操作飞书项目缺陷）

终端执行如下命令安装，期间终端会输出授权地址或二维码，用浏览器打开授权地址或手机飞书扫码，使用接收 bug 的飞书账号完成授权，授权完成后回到终端，等待安装命令结束

```shell
npx -y @lark-project/meegle@latest install \
  --host project.feishu.cn \
  --device-code \
  --lang zh
```

终端执行如下命令查看版本

```shell
meegle --version

# 期望输出：
# 1.0.24
```

终端执行如下命令查看飞书账号授权状态

```shell
meegle auth status --format json

# 期望输出：
# {
#   "authenticated": true,
#   "expires_in_minutes": 82,
#   "host": "project.feishu.cn"
# }
```

如果将来飞书授权过期了，终端执行如下命令扫码或获取 URL 重新授权

```shell
meegle auth login --host project.feishu.cn --device-code
```

注意：安装完成后，确认下本机上有没有成功安装 meegle 这个 Skill

![](img/3fca6bd1fb7082cefaa2b731f50caefe793c4120.png)

## 二、Codex 里一键改 BUG

我们编写了一个 bug-fix Skill，以便直接在 Codex 里通过显示调用或意图识别的方式触发这个 Skill 来发起一键改 BUG。

![](img/7c6f786e8ecb599bc1c283a3d2678f1705d9545c.png)

![](img/085b7aef24f84f88d481b60b3f01f0173b515ff1.png)
