# 2048 Game

基于 uni-app x 框架开发的 2048 小游戏，使用 UTS (Uni TypeScript) 编写。

## 功能特性

- 经典 2048 游戏玩法，支持 4x4 标准棋盘
- 手势滑动控制方块移动和合并
- 实时分数统计和最高分记录
- 游戏状态自动保存和恢复
- 撤消上一步操作
- 棋盘大小切换（3x3/4x4/5x5/6x6）
- 深色/浅色主题切换
- 计时功能

## 开发环境

- HBuilderX
- uni-app x 框架

## 技术栈

- **框架**: uni-app x
- **语言**: UTS (Uni TypeScript)
- **UI**: uvue
- **平台**: Android

## 项目结构

```
2048/
├── pages/
│   └── index/
│       └── index.uvue     # 游戏主界面
├── utils/
│   ├── gameLogic.uts      # 游戏核心逻辑
│   └── storage.uts        # 数据持久化
├── static/                # 静态资源
├── App.uvue               # 应用主组件
├── main.uts               # 应用入口
├── manifest.json          # 应用配置
├── pages.json             # 页面路由
└── uni.scss               # 全局样式
```

## 开发指南

1. 使用 HBuilderX 打开项目
2. 运行到 Android 设备或模拟器
3. 在真机上测试游戏功能

## 构建发布

在 HBuilderX 中选择「发行」->「原生 App-云打包」进行打包。
