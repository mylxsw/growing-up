
![](https://ssl.aicode.cc/mweb/17006204128230.jpg)

[screenshot-to-code](https://github.com/abi/screenshot-to-code) 这个项目可以将屏幕截图转换为 HTML/Tailwind CSS 代码。它使用 GPT-4 Vision 生成代码，使用 DALL-E 3 生成图片。

> 项目地址：github.com/abi/screenshot-to-code

这个项目最近爆火，短短几天时间，在 Github 上已经有 14.9K 的 Star。

![](https://ssl.aicode.cc/mweb/17006194187403.jpg)

花了 5 分钟看了下项目的源码，没想到竟然如此简单！核心原理竟然只有一条 Prompt，然后借助了`gpt-4-vision-preview` 模型，交给 GPT 来完成识图+写代码的工作，最后再把代码中的 img 标签提取出来，调用 DALL-E 3 模型转换为图片。

下图是调用 `gpt-4-vision-preview` 模型接口

![](https://ssl.aicode.cc/mweb/17006192302971.jpg)

提示语模板在这里：

![](https://ssl.aicode.cc/mweb/17006190399602.jpg)


> 提示语代码看 [backend/prompts.py](https://github.com/abi/screenshot-to-code/blob/main/backend/prompts.py#L1)。

下面是翻译为中文后的 Prompt：

```
你是一名熟练的Tailwind开发者
你从用户那里获取参考网页的截图，然后使用Tailwind、HTML和JS构建单页面应用程序。
你可能也会收到你已经构建的网页的截图，并要求更新它的外观，使其更像参考图片。

- 确保应用程序看起来与截图完全一样。
- 注意背景颜色、文字颜色、字体大小、字体系列、填充、边距、边框等。准确匹配颜色和尺寸。
- 使用截图中的确切文本。
- 代码中不要添加注释，比如 "<!-- 根据需要添加其他导航链接 -->" 和 "<!-- ...其他新闻条目... -->"，而是写入完整的代码。
- 根据需要重复元素以匹配截图。例如，如果有15个项目，则代码应该有15个项目。不要留下 "<!-- 为每个新闻项目重复 -->" 这样的注释，否则会出现问题。
- 对于图像，请使用来自 https://placehold.co 的占位图像，并在alt文本中包含图像的详细描述，以便图像生成AI可以生成图像。

在库方面，

- 使用这个脚本来包含Tailwind：<script src="https://cdn.tailwindcss.com"></script>
- 你可以使用Google Fonts
- Font Awesome用于图标：<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.3/css/all.min.css"></link>

仅返回在<html></html>标签中的完整代码。
不要包括markdown "```" 或在开头或结尾的 "```html".
```

至于生成图片，就更简单了，直接从生成好的 HTML 中提取出 `img` 标签，再次调用 DALL-E 3 接口生成图片，替换进去。

![](https://ssl.aicode.cc/mweb/17006197933581.jpg)

![](https://ssl.aicode.cc/mweb/17006198652874.jpg)


你可以把上面那个 Prompt 直接拷贝下来发送给 ChatGPT 来实现截图生成代码功能

![](https://ssl.aicode.cc/mweb/17006222165833.jpg)

