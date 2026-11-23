# v7.6 · 点击放大后「复制图片」（2026-11-23）

md5 `8a29052f098dd365298aeadc34e32e87` · 2111 行 · 92KB

作者要求：「复制到剪贴板 放到点击放大里」。**检测/裁切/像素算法一行未动**，只加视图层。

## 改了什么

点 A4 预览 → 全屏看图 → 底部一条工具栏，一个「复制图片」按钮，
把**整张 A4**（2480×3508 PNG，1.7MB）写进系统剪贴板，可直接粘到微信/Word。

`#fs` 从 `align-items:center` 改成竖排 column：图片 + 工具栏 + 「点空白处关闭」。
图片 `max-height:calc(100% - 92px)` 给工具栏留位，否则手机上按钮会被顶出屏幕。

## 三个必须踩对的点

1. **只能写 PNG**。Chrome 的剪贴板只接受 `image/png`，写 JPEG 直接抛错。
   预览大图本身是 JPEG（体积小），复制时另走 `buildSheet().toBlob(...,"image/png")`。
2. **Safari 的 write() 必须在用户手势的同一 tick 里发起**，而且 `ClipboardItem` 的值可以是
   Promise。所以顺序是「同步构造 ClipboardItem 并调 write → 把 toBlob 的 Promise 填进去」，
   绝对不能先 `await toBlob()` 再 write，否则 Safari 抛 NotAllowedError。
3. **提示条层级**。`#toast` 原 z-index 60 < `#fs` 80，复制成功的提示会被全屏遮罩盖住，
   用户看不到任何反馈。改成 90，并在全屏时用 `body.fsopen` 把它挪到**顶部**——
   底部 84px 的位置正好压在「复制图片」按钮上。

## 顺带修的老 bug

`#toast` 只写了 `left:50%` 没写 `right`，fixed 定位的可用宽度只有半个视口，
`max-width:min(86vw,420px)` 成了摆设 —— 文案被硬折行（实测 390 宽下可用宽度只有 195px）。
加 `width:max-content` 让它先按内容撑开，再交给 max-width 兜上限。这个问题从 v6 就在。

## 交互细节

- 点按钮**不**关闭全屏（`e.target.closest(".fsbtn")` 提前返回），点空白才关
- 桌面端 `Esc` 也能关
- 复制中按钮 disabled，完成/失败都恢复
- 浏览器不支持剪贴板写入（老 Safari、非 HTTPS）→ 提示「改用保存图片」而不是静默失败

## 验证（Chromium，390×844 / 1280×900）

- 剪贴板原本是 `SENTINEL` 文本，点击后变成 `image/png 1709611B 2480×3508` —— 确实被覆盖
- 复制耗时 ~150ms（含 toBlob 与 write），按钮 disabled 期间不误触
- 提示条落在 `[16,56]`，按钮在 `[647,691]`，**不重叠**；`zToast 90 > zFs 80`
- 抹掉 `navigator.clipboard` 走降级 → 提示正确、按钮恢复可用；恢复后仍能复制 PNG
- 其它长文案 toast 未溢出（84~306 / 视口 390）
- 控制台 **0 报错**

## 已知未做

复制的是**整张 A4**。若作者需要「只复制某一张卡」（比如只发人像面），
可以在卡片行再加一个复制按钮 —— 已问，等回复。
