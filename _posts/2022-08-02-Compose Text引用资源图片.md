---
layout: post
title: Compose Text引用PIC
subtitle: 如何像原生开发一样在Text中使用富文本
categories: android
tags: compose 技巧 
---

定义`inlineContent`

```kotlin
val inlineContentMap = mapOf(
	"testId" to InlineTextContent(
    	Placeholder(50.sp, 25.sp, PlaceholderVerticalAlign.Center)
    ){
        Image(painter = painterResource(id = R.drawable.xxx), contentDescription = null)
    }
)
```

Palceholder参数：(可能有误)

- width: 占位宽
- height: 占位高
- placeholderVerticalAlign: 占位空间内部组件相对于占位空间的位置

在Text中设置

```kotlin
Text(text = buildAnnotatedString{
    appendInlineContent(id = testId)
}, inlineContent = inlineContentMap)
```

