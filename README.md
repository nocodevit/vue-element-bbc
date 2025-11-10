# Vue Element 隐私政策设置面板

该示例项目使用 **Vue 3 + Vite + Element Plus** 搭建，提供一个“隐私政策设置”页面，支持默认策略与自定义 Markdown 编辑，同时在左侧模拟 H5 登录页预览，点击登录页底部的 `Privacy Policy` 可以在手机预览中查看最终策略内容。

## 功能概览

- 顶部 Tab 切换不同设置页，隐私政策页内置左右两栏布局：左侧手机预览、右侧设置表单。
- “自定义策略”开关关闭时，使用系统默认策略，不展示编辑区域。
- 开启自定义后：
  - 提供 Markdown 编辑器工具栏（支持 H1~H3、加粗、斜体、段落）。
  - 文本域下方提示自定义仅支持设置一种语言。
  - 保存按钮在修改内容后才可点击。
  - 手机预览会实时展示“隐私政策”页面：可在登录页与策略页之间切换。
- 页面采用统一的字体、色彩与阴影风格，符合 Element Plus 设计语言。

## 快速开始

```bash
# 安装依赖
npm install

# 启动开发环境
npm run dev

# 构建生产包
npm run build
```

默认情况下，Vite 会在浏览器打开 `http://localhost:5173/`，即可看到隐私政策设置页面。

## 目录结构

```
vue-element-bbc/
├── src/
│   ├── App.vue                  # 根组件，挂载隐私政策页面
│   ├── main.ts                  # 入口文件，注册 Element Plus
│   ├── assets/styles/global.css # 项目全局基础样式
│   └── views/
│       └── PrivacyPolicySetting.vue # 隐私政策设置主体页面
├── index.html                   # Vite HTML 模板
├── package.json                 # 项目依赖与脚本
├── tsconfig*.json               # TypeScript 配置
├── vite.config.ts               # Vite 配置
└── README.md                    # 项目说明
```

## 备注

- 由于示例未接入后端接口，`保存` 按钮当前通过 `setTimeout` 模拟异步，实际对接时可替换成真实 API 请求。
- 如果你使用其他包管理器（如 `pnpm`、`yarn`），将命令替换成对应语法即可。
- 项目已包含 `.gitignore`，默认忽略 `node_modules`、构建产物、常见编辑器配置等文件。

如需扩展更多设置页或接入真实数据，请在此基础上继续开发。祝使用愉快！


