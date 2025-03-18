---
title: Seven-Flow一个用于OA工作流程图设计的轻量级 javascript 插件
date: 2019-06-04 16:20:08
tags:
---


# Seven-Flow


Seven-Flow 是一个轻量级的 javascript 流程设计器，可用于OA工作流程图设计等

### 示例

> **基本**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8088b0caf2d8e4b31ab888bac1b17244.gif)

> **编辑节点**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1e047895fbe37e24132fd784b39a7d31.gif)

> **编辑线段**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0d1f2f6a400cb79c13cde6e9f70b17cd.gif)

### 安装使用
下载 [Seven-Flow](https://gitee.com/lujiayi/seven-flow/releases/v1.0)

引入seven-flow
```
<link rel="stylesheet" href="seven-flow.css">
<script src="seven-flow.js"></script>
```
容器
```
<div id="seven_flow"></div>
```
配置并初始化
```
var options = {
    page: {
        width: 1000,    // 画布宽度
        height: 600     // 画布高度    
    },
    node: {     // 节点配置
       	//  width: 120,     默认120
        //  height: 60,     默认60
        onConnect: (node) => {},    // 点击连接触发
        onSetting: (node) => {},    // 点击设置触发
        onClose: (node) => {}       // 点击删除触发
    },
    path: {     // 线段配置
        // stroke: '#888',     线段颜色
        onSetting: (path) => {}     // 点击设置触发
    },
    tools: {    // 工具栏配置
        saveFn: (e) => {},          // 点击确定触发
    },
    imgPath: './assets/images/'     // 图片资源路径
    editable: true		 // 是否可编辑
}

window.onload = () => sf.init('seven_flow', options);
```
数据回显
```
var data = {
    nodes: [...,
        {
            "id":"d152a250-869a-11e9-a15a-57fd9076db4e",
            "name":"节点",
            "type":"node",
            "attr":{...},
            "x":394,
            "y":49,
            "width":120,
            "height":60
        }
    ],
    path: [...,
        {
            "id":"fad046f0-869a-11e9-a15a-57fd9076db4e",
            "name":"开始-节点",
            "type":"path",
            "from":"d1134f60-869a-11e9-a15a-57fd9076db4e",
            "to":"d152a250-869a-11e9-a15a-57fd9076db4e",
            "reverse":"0",
            "lineType":"solid"
        }
    ]
}
sf.recover(data);
```