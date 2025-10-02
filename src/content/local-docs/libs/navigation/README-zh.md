# @gravity-ui/navigation &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/navigation)](https://www.npmjs.com/package/@gravity-ui/navigation) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/navigation/.github/workflows/ci.yml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/navigation/actions/workflows/ci.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/navigation/)

### 侧边栏头部导航 &middot; [预览 →](https://preview.yandexcloud.dev/navigation/)

![](docs/images/showcase.png)

## 安装

```bash
npm install @gravity-ui/navigation
```

确保在您的项目中安装了所需的 peer dependencies

```bash
npm install --dev @gravity-ui/uikit@^6.15.0 @gravity-ui/icons@2.2.0 @gravity-ui/components@3.0.0 @bem-react/classname@1.6.0 react@^18.0.0 react-dom@18.0.0
```

## 示例沙箱

基础版
https://codesandbox.io/p/devbox/navigation-demo-simple-x9k5sd

高级版
https://codesandbox.io/p/devbox/recursing-dawn-6kc9vh

## 2025 年路线图

1. 支持 SSR
2. 在 [Gravity UI](https://gravity-ui.com/ru/components/navigation/aside-header) 中添加更多文档和示例
3. 支持在 UIKit 主题工具中使用 Navigation
4. 统一 subheaderItem、menuItem 和 footerItem 的 API

## 组件

- [AsideHeader](https://github.com/gravity-ui/navigation/tree/main/src/components/AsideHeader/README.md)
  - [AllPagesPanel](https://github.com/gravity-ui/navigation/tree/main/src/components/AllPagesPanel/README.md)
  - PageLayout
- [PageLayoutAside](https://github.com/gravity-ui/navigation/tree/main/src/components/AsideHeader/README.md)
- AsideFallback
- FooterItem
- [Logo](https://github.com/gravity-ui/navigation/tree/main/src/components/Logo/Readme.md)
- [Drawer](https://github.com/gravity-ui/navigation/tree/main/src/components/Drawer/README.md)
- [DrawerItem](https://github.com/gravity-ui/navigation/blob/main/src/components/Drawer/README.md#draweritem-props)
- [MobileHeader](https://github.com/gravity-ui/navigation/tree/main/src/components/MobileHeader/README.md)
- MobileHeaderFooterItem
- MobileLogo
- [HotkeysPanel](https://github.com/gravity-ui/navigation/tree/main/src/components/HotkeysPanel/README.md)
- [Footer](https://github.com/gravity-ui/navigation/tree/main/src/components/Footer/README.md)
- [MobileFooter](https://github.com/gravity-ui/navigation/tree/main/src/components/Footer/README.md)
- [ActionBar](https://github.com/gravity-ui/navigation/tree/main/src/components/ActionBar/README.md)
- [Settings](https://github.com/gravity-ui/navigation/tree/main/src/components/Settings/README.md)

## CSS API

用于主题化 Navigation 的组件