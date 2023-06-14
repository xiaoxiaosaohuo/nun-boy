---
title: How to highlight a code block
date: 2023-06-14T22:13:05.274Z
description: ""
image: "images/posts/js-code.png"
categories: ["Tech"]
authors: ["siven"]
tags: ["Tech",]
draft: false
---

Yesterday, I tried to render remote markdown content in Astro, at first I used the code in Astro doc, which use `marked` to parse markdown content. However, I couldn't find an approach to highlight the code block.

At night, I browsed information on internet, the ` remark` came into me, and followed the `rehype`, then I found `unified`, this is an awesome project,it gives me a big help.

Today, I'll write this post to record some approaches to highlight code, in the following days, I will cover some basic principle under the hood.

## How to highlihgt code with rehype-highlight

```
import {unified} from 'unified'
import remarkParse from 'remark-parse'
import remarkGfm from 'remark-gfm'
import remarkRehype from 'remark-rehype'
import rehypeStringify from 'rehype-stringify'
import rehypeHighlight from 'rehype-highlight'
import rehypeSanitize,{defaultSchema} from 'rehype-sanitize'

const markdownToHtml = async (markdown)=>{
   
    const result = await unified()
    .use(remarkParse,{fragment:true})
    .use(remarkRehype)
    .use(remarkGfm)
    .use(rehypeHighlight,{detect:true,ignoreMissing:true})
    .use(rehypeSanitize)
    .use(rehypeStringify)
    .process(markdown)
    return result.toString()
}

export default markdownToHtml
```

when you do this, you'll find, oh, shit, it does not have any effect, just as before, wait..., wait..., you can open the Devtool,inspect the element, you'll find some code words have been add a class, such as `hljs-title`,`hljs-keyword`,

now, we can add our custom css style to pretty the code.

This is a theme that I find.

```
code.hljs {
  color: #cdd6f4;
  background: #1e1e2e
}

code .hljs-keyword {
  color: #cba6f7
}

code .hljs-built_in {
  color: #f38ba8
}

code .hljs-type {
  color: #f9e2af
}

code .hljs-literal {
  color: #fab387
}

code .hljs-number {
  color: #fab387
}

code .hljs-operator {
  color: #94e2d5
}

code .hljs-punctuation {
  color: #bac2de
}

code .hljs-property {
  color: #94e2d5
}

code .hljs-regexp {
  color: #f5c2e7
}

code .hljs-string {
  color: #a6e3a1
}

code .hljs-char.escape_ {
  color: #a6e3a1
}

code .hljs-subst {
  color: #a6adc8
}

code .hljs-symbol {
  color: #f2cdcd
}

code .hljs-variable {
  color: #cba6f7
}

code .hljs-variable.language_ {
  color: #cba6f7
}

code .hljs-variable.constant_ {
  color: #fab387
}

code .hljs-title {
  color: #89b4fa
}

code .hljs-title.class_ {
  color: #f9e2af
}

code .hljs-title.function_ {
  color: #89b4fa
}

code .hljs-params {
  color: #cdd6f4
}

code .hljs-comment {
  color: #585b70
}

code .hljs-doctag {
  color: #f38ba8
}

code .hljs-meta {
  color: #fab387
}

code .hljs-section {
  color: #89b4fa
}

code .hljs-tag {
  color: #a6adc8
}

code .hljs-name {
  color: #cba6f7
}

code .hljs-attr {
  color: #89b4fa
}

code .hljs-attribute {
  color: #a6e3a1
}

code .hljs-bullet {
  color: #94e2d5
}

code .hljs-code {
  color: #a6e3a1
}

code .hljs-emphasis {
  color: #f38ba8;
  font-style: italic
}

code .hljs-strong {
  color: #f38ba8;
  font-weight: bold
}

code .hljs-formula {
  color: #94e2d5
}

code .hljs-link {
  color: #74c7ec;
  font-style: italic
}

code .hljs-quote {
  color: #a6e3a1;
  font-style: italic
}

code .hljs-selector-tag {
  color: #f9e2af
}

code .hljs-selector-id {
  color: #89b4fa
}

code .hljs-selector-class {
  color: #94e2d5
}

code .hljs-selector-attr {
  color: #cba6f7
}

code .hljs-selector-pseudo {
  color: #94e2d5
}

code .hljs-template-tag {
  color: #f2cdcd
}

code .hljs-template-variable {
  color: #f2cdcd
}

code .hljs-diff-addition {
  color: #a6e3a1;
  background: rgba(166, 227, 161, .15)
}

code .hljs-diff-deletion {
  color: #f38ba8;
  background: rgba(243, 139, 168, .15)
}
```
It looks good, but some word can not be highlight, do we have a better approach to fix this?.

## rehype-pretty-code

this is the perfect way to highlight code.

just as before, we replace `rehype-highlight` 

```
import {unified} from 'unified'
import remarkParse from 'remark-parse'
import remarkGfm from 'remark-gfm'
import remarkRehype from 'remark-rehype'
import rehypeStringify from 'rehype-stringify'
import rehypeHighlight from 'rehype-highlight'
import rehypeSanitize,{defaultSchema} from 'rehype-sanitize'
import rehypePrettyCode from 'rehype-pretty-code';

const markdownToHtml = async (markdown)=>{
   
    const result = await unified()
    .use(remarkParse,{fragment:true})
    .use(remarkRehype)
    .use(remarkGfm)
    .use(rehypePrettyCode, {
        theme: 'one-dark-pro',
        keepBackground: true,
      // See Options section below.
    })
    // .use(rehypeHighlight,{detect:true,ignoreMissing:true})
    // .use(rehypeSanitize)
    .use(rehypeStringify)
    .process(markdown)
    return result.toString()
}

export default markdownToHtml
```

then you'll get this beautiful highlight effect.

![hiking1](/nun-boy/images/posts/js-code.png)


#### Refrences
- https://shiki.matsu.io/
- https://github.com/rehypejs/rehype-highlight
- https://github.com/highlightjs/highlight.js/blob/main/src/styles/github-dark-dimmed.css
- https://github.com/highlightjs/highlight.js
- https://remark.js.org/
- https://rehype-pretty-code.netlify.app/
- https://prismjs.com/