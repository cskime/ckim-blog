---
title: "Simple Way to Add Image to Hugo Post"
date: 2022-11-11T21:27:07+09:00
draft: false
summary: "How to use image in post."
tags: ['blog']
---

First, add the image to `static/images` directory. This is global directory for static resources.

You can simply use markdown syntax to add image in your post.

```markdown
![alt](/images/filename.png)
```
![alt](/images/avatar.png)

But, if you want to resize image, you should use **['shortcode'](https://gohugo.io/content-management/shortcodes)**, `figure`. (In my case, `<img />` tag also didn't work.)

```
\{\{< figure src="/images/filename.png" >\}\}
```
{{< figure src="/images/avatar.png" >}}

```
\{\{< figure src="/images/filename.png" width="100px" >\}\}
```
{{< figure src="/images/avatar.png" width="100px" >}}

```
\{\{< figure src="\images/filename.png" width="40%" >\}\}
```
{{< figure src="/images/avatar.png" width="40%" >}}