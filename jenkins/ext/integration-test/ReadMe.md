# 建立Jenkins Integration Test

当一个新的pull request被发起的时候，希望执行代码测试，测试提交的代码是否ok，此时可以选择TravisCI，也可以选择Github Action

- TravisCI对于github open source public repo是免费的，但是如果github repo是私有的，那么只能build 100次，有这样的限制后期就迁移到了Github Action
- 另外如果github repo集成TravisCI，可以选择指定哪个repo install github apps，而不需要在全局github/settings/apps中安装github app
- 初期使用的是[Travis](https://travis-ci.com/account/repositories)来做Github 的CI，关于Travis的配置，本例中使用的是Github直接与Travis CI互联，更多关于使用Travis，查看其官网，以及`ext/Using TravisCI xxx.pdf`
- 后期直接将Github Integration Test迁移到Github Action
- 已创建的[TravisCI](https://travis-ci.com/account/repositories),使用HuangMarco账户登录

## 使用Git hub Action 配置integration test

先阅读[GitHub Actions 快速入门](https://docs.github.com/cn/actions/quickstart)

在阅读[GitHub Actions 简介](https://docs.github.com/cn/actions/learn-github-actions/introduction-to-github-actions)

参考[从 Travis CI 迁移到 GitHub Actions](https://docs.github.com/cn/actions/learn-github-actions/migrating-from-travis-ci-to-github-actions)


## 拓展

[Installing GitHub Apps](https://docs.github.com/en/developers/apps/managing-github-apps/installing-github-apps)
[从 Travis CI 迁移到 GitHub Actions](https://docs.github.com/cn/actions/learn-github-actions/migrating-from-travis-ci-to-github-actions)
[Setting up continuous integration using workflow templates](https://docs.github.com/en/actions/guides/setting-up-continuous-integration-using-workflow-templates)
[非必须CI-applitools](https://applitools.com/blog/applitools-eyes-github-integration-how-to-visually-test-every-pull-request/)
[github-bot-usr-github-pull-request-plugin-jenkins](https://github.com/jenkinsci/ghprb-plugin/blob/master/README.md)
