---
name: create-component
description: 基于模板创建通用 UI 组件（非弹窗、非页面），包含 JSX、LESS 和素材目录。当用户要求创建组件、新增组件、添加按钮组、创建 UI 模块时使用。
---

# 创建通用组件

基于项目模板，创建可复用的 UI 组件（如按钮组、导航栏、卡片等），包含 JSX、LESS 文件和素材目录。

## 流程

### 1. 确定组件名称

组件名使用 **PascalCase**（如 `NavBtnGroup`、`UserCard`、`TabBar`）。

### 2. 分析设计稿

如果用户提供了设计截图：
- 识别组件中包含的元素（按钮、图标、文本等）
- 确定元素的排列方式（横向/纵向/绝对定位）
- 为每个需要图片素材的元素规划语义化命名

### 3. 创建组件文件

在 `src/components/{ComponentName}/` 下创建两个文件：

**{ComponentName}.jsx** — 基于以下模板，将所有 `__ComponentName__` 替换为实际名称：

```jsx
'use strict';

import React from 'react';
import './__ComponentName__.less';

class __ComponentName__ extends React.Component {
  render() {
    // 根据设计稿添加 props 解构和元素
    return (
      <div className="__ComponentName__">
        {/* 根据设计稿添加子元素 */}
        {/* 图片素材元素用 <span className="xxx" /> */}
        {/* 可点击元素绑定 props 回调: onClick={this.props.onXxx} */}
      </div>
    );
  }
}

export default __ComponentName__;
```

**{ComponentName}.less** — 基于以下模板：

```less
@import "../../res.less";

.__ComponentName__ {
  // 容器尺寸和定位，根据实际情况调整
  display: flex;
  position: absolute;

  // 每个子元素使用 .webpBg() 引用素材
  // 尺寸先用占位值，等素材放好后用 sips 获取实际像素尺寸替换
  .elementName {
    width: 200px;
    height: 80px;
    .webpBg("__ComponentName__/elementName.png");
  }
}
```

### 4. 创建素材目录

使用 Shell 命令创建 `src/assets/{ComponentName}/` 目录：

```bash
mkdir -p src/assets/{ComponentName}
```

### 5. 告知用户放入素材

列出需要放入的素材文件清单（文件名 + 说明），等待用户放好。

### 6. 素材放好后更新代码

用户确认素材放好后：

1. **扫描素材**：用 Glob 扫描 `src/assets/{ComponentName}/` 下所有图片
2. **查看图片**：用 Read 工具查看每张图片内容，确认用途
3. **获取尺寸**：用 `sips -g pixelWidth -g pixelHeight` 获取每张图片的像素尺寸
4. **更新 LESS**：将占位尺寸替换为实际像素值

## Props 设计规范

- 可点击元素通过 props 传入回调函数，命名为 `onXxx`（如 `onPrev`、`onNext`、`onClose`）
- 组件不直接依赖 store，保持纯展示 + 回调的模式，由父组件注入逻辑
- 如需响应 store 数据，由父组件传入 props 或在组件内按需引入

## 样式规范

- 所有图片素材通过 `.webpBg("{ComponentName}/{filename}")` 引用
- 宽高使用图片的实际像素值（通过 sips 获取）
- 容器宽度参考 750px 设计稿宽度
- 元素水平居中时 `left = (750 - width) / 2`
