---
date: 2026-04-27T20:00:00+08:00
title: "CoolCassette：AI 帮你的 Walkman 生成磁带皮肤 🎵"
description: "把你的专辑封面变成 Walkman 上的复古磁带皮肤，下载应用，三步搞定。"
tags: ["walkman", "ai", "music"]
series: []
featured: true
---

偶然刷到 [Wampy](https://github.com/unknown321/wampy)，一个让 Walkman 显示复古磁带界面的自制插件。用了一段时间后我发现，Wampy 支持自定义每张专辑的磁带皮肤——但手动做一张实在麻烦：要找封面、调尺寸、画卷轴动画、转换格式。

所以 CoolCassette 出现了！把 Walkman 插上电脑，它会读取里面的音乐，根据专辑封面自动生成磁带皮肤，然后直接装到设备上。整个过程点几下就完成了。不满意？重新抽奖！

<!--more-->

---

## 长什么样

想象一下，你打开 Walkman，播放一张 Radiohead 的专辑，屏幕上出现一盘磁带——外壳的颜色和质感跟专辑封面的氛围完美匹配，磁带的卷轴还在慢慢转动。换一张The Cure的专辑，磁带就变成了另一个样子。每一张专辑都有自己专属的皮肤。

![cassette](/images/coolcassette-guide-cassette.jpg)

当然你可以单纯用来浏览你的音乐收藏？！

![albums](/images/coolcassette-guide-screenshot.jpg)

---

## 下载

{{< tabgroup >}}
{{< tab name="macOS (Apple Silicon)" >}}

[下载 CoolCassette-v0.1.3-macos-arm64.tar.gz](https://github.com/y7ut/CoolCassette/releases/download/v0.1.3/CoolCassette-v0.1.3-macos-arm64.tar.gz)

下载后双击解压，把 `CoolCassette.app` 拖到「应用程序」文件夹即可。

首次打开时，macOS 可能会提示"无法验证开发者"——右键点击应用，选择「打开」，然后在弹出的对话框中再次点击「打开」就好了。

或者提示应用程序已经损坏, 需要前往应用所在目录，打开终端 terminal 输入

```sh
sudo xattr -dr com.apple.quarantine CoolCassette.app
```

{{< /tab >}}
{{< tab name="Windows" >}}

[下载 CoolCassette-v0.1.3-windows-amd64.zip](https://github.com/y7ut/CoolCassette/releases/download/v0.1.3/CoolCassette-v0.1.3-windows-amd64.zip)

下载后解压到任意文件夹，双击 `CoolCassette.exe` 运行。

首次运行时 Windows 可能弹出 SmartScreen 警告，点击「更多信息」→「仍要运行」即可。

{{< /tab >}}
{{< /tabgroup >}}

---

## 使用前需要准备的东西

### 1. ImageMagick

这是一个图片处理工具，CoolCassette 用它来裁剪封面和合成磁带图像。

{{< tabgroup >}}
{{< tab name="macOS" >}}

如果你装了 Homebrew，打开终端输入：

```bash
brew install imagemagick
```

{{< /tab >}}
{{< tab name="Windows" >}}

去 [ImageMagick 官网](https://imagemagick.org/script/download.php#windows) 下载安装包。安装时记得勾选 "Add to system PATH"。

{{< /tab >}}
{{< /tabgroup >}}

### 2. AI API Key

磁带图像是 AI 生成的，所以你需要一个 API Key

当前是使用的是Gemini NanoBanana 2 模型，每张大概 $0.05，非常便宜。
默认会通过 OpenRouter 的渠道来调用（悲报: 我的KEY已经被封禁了）

默认会读取系统环境变量中的OPENROUTER_API_KEY
不过也可以使用配置文件, 你可以在你的用户根目录创建一个`.coolcassette.json`文件
Windows: `C:\Users\你的用户名\.coolcassette.json`
Macos: `~/.coolcassette.json` 或 `/Users/你的用户名/.coolcassette.json`

macos 快速初始化

```sh
echo '{"api_key":"sk-or-...","provider":"openrouter"}' > ~/.coolcassette.json
```

推荐用 [OpenRouter](https://openrouter.ai)，注册账号后在设置页面创建一个 Key，以 `sk-or-` 开头。生成一张磁带皮肤大概5毛钱。


---

## 三步上手

### 第一步：连接 Walkman，配置目录

1. 用 USB 线把 Walkman 连到电脑
2. 打开 CoolCassette，点击右上角的 **Settings**
3. 配置以下内容：

| 设置项 | 填什么 |
|--------|--------|
| **Music Directory** | Walkman 上的音乐目录，例如 `E:\Music\` 你也可以顺便添加储存卡的音乐目录（macOS 下可能是 `/Volumes/WALKMAN/Music`） |
| **Wampy Directory** | Walkman 上的 Wampy 目录，通常是`E:\wampy\`（macOS 下可能是 `/Volumes/WALKMAN/wampy`） |


> **怎么找这些目录？** Walkman 连上电脑后，在文件管理器（macOS 是 Finder，Windows 是资源管理器）里打开 Walkman 的磁盘，你会看到 `Music` 和 `wampy` 这两个文件夹。CoolCassette 设置界面里有文件夹浏览功能，直接点击选择就行。（目前win 选择器有问题，需要手动输入目录地址，例如`E:\Music\`, `E:\wampy\`）


配置完成后，点击 Reload Library 应用会自动扫描你 Walkman 里的音乐，把专辑列表展示出来。(推荐使用MusicCenter 导入音乐，这样会自动匹配目录，注意 Coolcassette 扫描专辑时候，只支持按照文件夹分组的专辑目录)

不用我说，你一定知道在下载了新的专辑后，需要重新导入音乐库🥹

### 第二步：生成磁带皮肤

1. 在专辑列表里找到你想生成的专辑，点击进入
2. 点击 **Generate Preview** 按钮
3. 等待 10-30 秒，AI 会根据专辑封面生成磁带图像
4. 你可以预览效果——包括磁带外壳和转动的卷轴动画

### 第三步：发布到 Walkman

觉得效果满意的话，点击 **Publish** 按钮，CoolCassette 会自动把皮肤安装到你的 Walkman 上。

断开 USB，打开 Walkman 上的 Wampy 播放器，播放对应专辑——你的专属磁带皮肤就出现了。

> 如果生成效果不满意，可以再次点击 **Generate Preview**，AI 每次生成的结果都不一样，多试几次总能找到喜欢的。

---

## 常见问题

### 生成一张皮肤要花钱吗？

用 OpenRouter 的 Gemini NanoBanana 2 模型，每张大概 $0.05，非常便宜。

### 支持哪些音乐格式？

MP3、FLAC、WAV、M4A、M4B、AAC、MP4 都支持。封面会自动从音频文件的标签信息中提取。

### 封面是哪来的？

CoolCassette 会按这个顺序找封面：专辑文件夹里的 `cover.jpg`（或 png）→ 音频文件里嵌入的封面图。

### 支持哪些 Walkman？

所有装了 [Wampy](https://github.com/unknown321/wampy) 插件的 Sony NW 系列播放器。

---

## 项目信息

- **GitHub**: [y7ut/CoolCassette](https://github.com/y7ut/CoolCassette)
- **问题反馈**: [GitHub Issues](https://github.com/y7ut/CoolCassette/issues)
