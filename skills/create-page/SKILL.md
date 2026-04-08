---
name: create-page
description: 基于模板创建页面组件并自动注册到路由。当用户要求创建页面、新增 Page、添加新页面时使用。
---

# 创建页面组件

基于项目页面模板，创建新的页面组件并注册到 `src/app.jsx` 路由和 `src/utils/constants.js` 页面常量中。

## 流程

### 1. 确定页面名称

页面组件名使用 **PascalCase**（如 `DetailPage`、`SharePage`、`AgentPage`）。

### 2. 创建组件文件

在 `src/pages/{PageName}/` 下创建两个文件：

**{PageName}.jsx** — 基于以下模板，将所有 `__PageName__` 替换为实际名称：

```jsx
import React from 'react';
import { observer } from 'mobx-react';
import './__PageName__.less';
import store from '@src/store/index';

@observer
class __PageName__ extends React.Component {

  constructor(props) {
    super(props);
  }

  componentDidMount() {
  }

  render() {
    return (
      <div className="__pageName__">
        <span className="bg"></span>
      </div>
    );
  }
}

export default __PageName__;
```

> 注意：`className` 使用 **camelCase**（如 `detailPage`），组件名使用 **PascalCase**（如 `DetailPage`）。

**{PageName}.less** — 基于以下模板：

```less
@import "../../res.less";

.__pageName__ {
  width: 750px;
  height: 100vh;
  left: 0px;
  top: 0px;
  position: absolute;
  overflow-x: hidden;
  overflow-y: auto;
  .hideScrollbar();

  .bg {
    position: absolute;
    width: 750px;
    height: 1624px;
    left: 0px;
    top: 0px;
    // .webpBg("__PageName__/bg.jpg");
  }
}
```

### 3. 创建素材文件夹

在 `src/assets/{PageName}/` 下创建同名文件夹，用于存放页面的背景图等素材资源。

使用 Shell 命令创建：
```bash
mkdir -p src/assets/{PageName}
```

### 4. 注册页面

需要修改两个文件：

**src/utils/constants.js** — 在 `PAGE_MAP` 中添加页面常量：

```javascript
export const PAGE_MAP = {
  // ...已有页面...
  __UPPER_SNAKE__: "__camelCase__",
};
```

命名规则：
- key 使用 `UPPER_SNAKE_CASE`（如 `DETAIL_PAGE`）
- value 使用 `camelCase`（如 `detailPage`）

**src/app.jsx** — 添加 import 和路由注册：

1. 在 import 区域添加：
   ```jsx
   import __PageName__ from "@src/pages/__PageName__/__PageName__";
   ```

2. 在 `pageMap` 对象中添加：
   ```jsx
   [PAGE_MAP.__UPPER_SNAKE__]: <__PageName__ />,
   ```

## 注意事项

- 页面容器宽度固定 750px，背景高度默认 1624px
- 页面使用 `overflow-y: auto` 支持纵向滚动，配合 `.hideScrollbar()` 隐藏滚动条
- 背景图通过 `.webpBg()` 引用，素材放好后取消注释即可
- 页面间跳转通过 `store.changePage(PAGE_MAP.xxx, data)` 实现
