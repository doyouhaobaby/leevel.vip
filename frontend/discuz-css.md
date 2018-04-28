# Discuz CSS 属性书写顺序规范

以前研究 Discuz 的时候，发现它的 CSS 样式书写顺序挺好。按照元素模型由外及内，由整体到细节书写，大致分为五组。

```
位置：position,left,right,float
盒模型属性：display,margin,padding,width,height
边框与背景：border,background
段落与文本：line-height,text-indent,font,color,text-decoration,...
其他属性：overflow,cursor,visibility,...
```