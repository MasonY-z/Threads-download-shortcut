# Threads Video Download — iOS Shortcut

**English** | [中文](#中文说明)

Download videos from Threads straight to your iPhone with one tap from the Share Sheet. No third-party websites, no servers, no login — everything runs locally in Apple Shortcuts.

## How it works

Every public Threads post has an **embed page** (`.../post/<id>/embed`) that loads without logging in. The video URL is written directly in that page's HTML source as a `<source src="...">` tag. The shortcut:

1. Takes the link you share from the Threads app (usually a `threads.com/share/...` short link)
2. Expands it to the full post URL and strips the query string
3. Appends `/embed` and fetches the page
4. Extracts the `.mp4` URL with a regex and decodes `&amp;` → `&`
5. Downloads the video and saves it to a folder you choose

## Install

- **Option A — `.shortcut` file (recommended):** download `Threads-Download.shortcut` from the [**latest release**](https://github.com/MasonY-z/Threads-download-shortcut/releases/latest), open it on your iPhone (e.g. from the Files app), and tap **Add Shortcut**. The file is signed for **Anyone**, so no special settings are needed.
- **Option B — build it yourself:** follow the steps below (about 10 minutes)

> Built and tested on the latest iOS. Older iOS versions may fail to import it — use Option B in that case.

## Usage

1. In the Threads app, tap the **share icon** (paper plane) on a video post
2. Choose **Threads Download**
3. On first run, tap **Always Allow** for each permission prompt
4. Pick a folder in the save dialog (it remembers your last choice)
5. Tap **Open Folder** in the menu to jump to the Files app

You can also copy a post link and run the shortcut directly — it falls back to the clipboard.

## Build it yourself

Tap **Edit** on a new shortcut to skip the AI "Describe a shortcut" screen. In **ⓘ Details**, turn on **Show in Share Sheet**.

| # | Action | Settings |
|---|---|---|
| 1 | Receive | **Any** input from **Share Sheet**; if there's no input: **Get Clipboard** |
| 2 | Get URLs from Input | Input: **Shortcut Input** (variable) |
| 3 | Expand URL | URL: **URLs** (variable) |
| 4 | Replace Text | Find `/?(\?.*)?$`, replace with nothing, in **Expanded URL**; **Regular Expression: on** |
| 5 | Text | **Updated Text** (variable) immediately followed by `/embed` — no space |
| 6 | Get Contents of URL | URL: **Text**; Headers: `User-Agent` = iPhone Safari UA (see below) |
| 7 | Set Name | Set name of **Contents of URL** to `page.txt` |
| 8 | Match Text | Pattern `<(?:video\|source)[^>]*?\ssrc="([^"]+)"` in **Renamed Item** |
| 9 | Get Group from Matched Text | **Group At Index** `1` in **Matches** |
| 10 | Replace Text | Find `&amp;`, replace with `&`, in step 9's output; regex **off** |
| 11 | If | **Updated Text** (from step 10) **does not have any value** → Show Alert `No video found` → Stop This Shortcut → End If |
| 12 | Repeat for Each | Items: **Updated Text** from step 10 |
| 13 | ↳ Get Contents of URL | URL: **Repeat Item** |
| 14 | ↳ Set Name | **Contents of URL** → `Threads_` + **Current Date** (Custom format `yyyyMMdd_HHmmss`) + `.mp4` |
| 15 | ↳ Save File | **Renamed Item**; **Ask Where to Save: on** |
| 16 | End Repeat | — |
| 17 | Choose from Menu | Prompt `Download Finished ✅`; **Open Folder** → Open URLs `shareddocuments://`; **OK** → nothing |

In step 8, the pattern contains a plain `|` — the backslash above is only for table formatting.

User-Agent value for step 6:

```
Mozilla/5.0 (iPhone; CPU iPhone OS 18_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/18.0 Mobile/15E148 Safari/604.1
```

### Pitfalls we hit (read this if it doesn't work)

- **Variables vs. typed text.** A connected variable shows an **icon** before its name. Light-blue text with no icon is a placeholder, and black text is literal text you typed. If `Shortcut Input` in step 2 is typed text instead of the variable, every later step is empty.
- **The `page.txt` rename (step 7) is essential.** Without it, Shortcuts treats the HTML as rich text and strips every tag, so the regex finds nothing.
- **Receive must be "Any".** Threads shares a rich web-page object; restricting input to URLs/Text filters it out.
- **Repeat must use step 10's Updated Text**, not "If Result" — the If block outputs nothing.
- **Two variables share the name "Updated Text".** Tap a variable → **Reveal Action** to check which step it comes from.
- **Placeholder "File" in Save File opens a file picker when tapped.** Long-press it, or delete and re-add the action directly below Set Name so it links automatically.

## Test on a Mac

The same logic as a one-liner — useful to check whether Threads has changed its page structure:

```bash
UA="Mozilla/5.0 (iPhone; CPU iPhone OS 18_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/18.0 Mobile/15E148 Safari/604.1"
POST="https://www.threads.com/@username/post/POST_ID"

URL=$(curl -sL -A "$UA" "$POST/embed" \
  | grep -oE '<source[^>]*src="[^"]+"' | head -1 \
  | sed -E 's/.*src="([^"]+)"/\1/; s/&amp;/\&/g')
curl -sL "$URL" -o threads_video.mp4 && open threads_video.mp4
```

## Limitations

- **Quality:** you get the single version Threads serves on the embed page, usually 720p or 1080p depending on the upload.
- **Expiring links:** video URLs contain an expiry (`oe=` parameter), so the shortcut always resolves them fresh.
- **Fragility:** this depends on Threads' embed page structure. If Meta changes it, the regex in step 8 may need updating — issues and PRs welcome.
- **Public posts only.** Private accounts and image-only posts won't work.
- `shareddocuments://` opens the Files app but can't target a specific folder. Mark your download folder as a **Favourite** so it's one tap away.

## Disclaimer

This project is not affiliated with, endorsed by, or connected to Meta or Threads. It only accesses publicly available content. Use it for personal use, respect creators' rights, and don't redistribute content without permission.

## License

MIT

---

## 中文说明

在 Threads App 里点一下分享，就能把视频下载到 iPhone。不经过第三方网站、不需要服务器、不用登录，全部在「快捷指令」里本地运行。

### 原理

每个公开的 Threads 帖子都有一个**嵌入页面**（`.../post/<帖子ID>/embed`），不登录也能访问，而且视频地址直接写在网页源码的 `<source src="...">` 里。快捷指令做的事情是：

1. 接收从 Threads 分享的链接（通常是 `threads.com/share/...` 短链接）
2. 展开成完整的帖子地址，并去掉 `?` 后面的参数
3. 在末尾加上 `/embed`，然后下载这个页面
4. 用正则提取 `.mp4` 地址，并把 `&amp;` 还原成 `&`
5. 下载视频，保存到你选择的文件夹

### 安装

- **方式 A：`.shortcut` 文件（推荐）**：从 [**最新 Release**](https://github.com/MasonY-z/Threads-download-shortcut/releases/latest) 下载 `Threads-Download.shortcut`，在 iPhone 上打开（比如从「文件」App 里点开），然后点 **Add Shortcut**。文件已签名为「任何人」可用，不需要改任何设置。
- **方式 B：自己搭建**：按照上面英文部分的步骤表操作，大约 10 分钟

> 在最新版 iOS 上制作和测试。旧版本 iOS 可能无法导入，这种情况请用方式 B。

### 使用方法

1. 在 Threads App 里，对视频帖子点**分享（纸飞机图标）**
2. 选 **Threads Download**
3. 第一次运行时，弹出的权限请求都选 **Always Allow**
4. 在保存窗口里选一个文件夹（它会记住上次的选择）
5. 在弹出的菜单里点 **Open Folder**，就会跳转到「文件」App

也可以先复制帖子链接，再直接运行快捷指令，它会自动读取剪贴板。

### 常见的坑

- **变量和手打文字的区别**：真正连上的变量，名字前面有**图标**；浅蓝色、没有图标的是占位文字；黑色的是你手打的普通文字。如果第 2 步的 `Shortcut Input` 是手打的文字而不是变量，后面每一步都会是空的。
- **改名成 `page.txt`（第 7 步）必不可少**：不改名的话，快捷指令会把 HTML 当成富文本处理，把所有标签都去掉，正则就什么都匹配不到。
- **Receive 必须选 Any**：Threads 分享出来的是网页对象，如果只允许 URLs 和 Text，它会被过滤掉。
- **Repeat 要用第 10 步的 Updated Text**，不能用 If Result，因为 If 这一整块不会输出任何内容。
- **有两个变量都叫 Updated Text**：点变量 → **Reveal Action**，可以确认它来自哪一步。
- **Save File 里的 File 占位符一点就会打开文件选择器**：长按它来选变量，或者删掉后重新添加到 Set Name 的正下方，让它自动连上。

### 局限

- **画质**：拿到的是 embed 页面提供的那个版本，一般是 720p 或 1080p，取决于原视频
- **链接会过期**：视频地址带有效期（`oe=` 参数），所以每次都要重新解析
- **可能失效**：这个方法依赖 Threads embed 页面的结构，Meta 改版后可能需要更新第 8 步的正则，欢迎提 issue 和 PR
- **只支持公开帖子**：私密账号和纯图片帖子不行
- `shareddocuments://` 只能打开「文件」App，不能指定具体文件夹。把下载文件夹设为**个人收藏（Favourite）**，就能一点进入

### 免责声明

本项目与 Meta 或 Threads 没有任何关联，也未获得其认可，只访问公开可见的内容。请仅供个人使用，尊重创作者的权益，未经许可请勿二次传播。
