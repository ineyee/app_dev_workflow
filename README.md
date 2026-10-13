## 【写 UI 阶段】写静态 UI

我们编写了一个 `static-ui` Skill，以便直接在 Codex 里通过显示调用或意图识别的方式触发这个 Skill 来实现 Figma 设计稿。

![](img/468ba1719ee297e0f21ea5dad57c9c9c36d2ad6d.png)

![](img/4973ce976e52b26357629fe9b81be8aba37f847f.png)

## 【写逻辑阶段】iOS、Android 逻辑互转（包含业务逻辑、UI 逻辑等）

我们编写了一个 `ios-to-android-plan` Skill，以便直接在 Codex 里通过显示调用或意图识别的方式触发这个 Skill 来根据 iOS 实现生成 Android 实施计划，另外一个 `android-to-ios-plan` Skill 同理。

* 如果没有具体的提示词，则会把当前会话内所有落地的需求生成安卓实施计划

* 如果有具体的提示词，则只把具体的逻辑生成安卓实施计划

![](img/12149061055fae4ec82d1ae54e8438d1338e6236.png)

![](img/73cab2ac165cc85011ad3bfea3d7fdbd2ad55cd9.png)

![](img/a4bc62e2b95bc8e5e847033eeae15a104eaa0197.png)

![](img/6f3508f617080d40aafaff03ab19688f680b3cc5.png)

## 【提测阶段】iOS、Android 提测蒲公英 + 企微通知

我们编写了一个 `pgyer-ios` Skill，以便直接在 Codex 里通过显示调用或意图识别的方式触发这个 Skill 来发起一键打包 + 上传蒲公英 + 通知企微，而不用每次都去终端里敲命令，另外一个 `pgyer-android` Skill 同理。

![](img/208acb7b213dc4e3aba8356214e3070b718e5ae6.png)

![](img/37518223db1295bef8ce8cc1097bb3c6aa842613.png)
