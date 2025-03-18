---
title: 解决Angular Kendo UI 导出PDF中文乱码
date: 2018-07-02 16:16:59
tags:
---
在使用 [Kendo UI for Angular](https://www.telerik.com/kendo-angular-ui/) 的 [PDFExportComponent](https://www.telerik.com/kendo-angular-ui/components/pdfexport/api/PDFExportComponent/) 组件时，如果内容是中文则导出PDF将会出现乱码，网上许多人认为是官方不支持中文，但其实不然，阅读文档

**The PDF Export component supports the Unicode standard only if the fonts you provide contain glyphs for the referenced Unicode characters**

发现导致乱码的问题是该组件只支持Unicode标准的字体，官方默认字体是 DejaVu Sans，这字体一看就不是好东西，胆敢不支持我大天朝的语言。找到乱码的原因了，那接下来就是换了这个字体，官方提供了两种方式：
    
- @font-face 指定
    
[Microsoft Yahei 字体下载](https://ufonts.com/download/microsoft-yahei/181072.html)，下载完成后将.ttf文件存放在项目下，然后在@font-face 下的 src 指定到该文件，如下
```
@font-face {
  font-family: 'DejaVu Sans';
  src: url('../../Microsoft Yahei.ttf') format('truetype');
}
```

- defineFont 方法(推荐)

同样下载 [Microsoft Yahei 字体](https://ufonts.com/download/microsoft-yahei/181072.html)，设置如下：

```
import { defineFont } from '@progress/kendo-drawing/pdf';

...

@ViewChild('pdf') pdf: PDFExportComponent;

public print(): void {
    ...
    defineFont({
        'Microsoft Yahei': '../../Microsoft Yahei.ttf'
    });//也可以在 ngInit() 中设置
    
    this.pdf.saveAs('example.pdf');
}
```
OK，可以愉快的导出中文了~