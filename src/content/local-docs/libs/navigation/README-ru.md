# @gravity-ui/navigation &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/navigation)](https://www.npmjs.com/package/@gravity-ui/navigation) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/navigation/.github/workflows/ci.yml?branch=main&label=CI&logo=github)](https://github.com/gravity-ui/navigation/actions/workflows/ci.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/navigation/)

### Навигация в боковой панели с заголовком &middot; [Предпросмотр →](https://preview.yandexcloud.dev/navigation/)

![](docs/images/showcase.png)

## Установка

```bash
npm install @gravity-ui/navigation
```

Убедитесь, что в вашем проекте установлены зависимости peer:

```bash
npm install --dev @gravity-ui/uikit@^6.15.0 @gravity-ui/icons@2.2.0 @gravity-ui/components@3.0.0 @bem-react/classname@1.6.0 react@^18.0.0 react-dom@18.0.0
```

## Примеры в песочницах

Простой
https://codesandbox.io/p/devbox/navigation-demo-simple-x9k5sd

Расширенный
https://codesandbox.io/p/devbox/recursing-dawn-6kc9vh

## Дорожная карта 2025

1. Поддержка SSR
2. Добавить больше документации и примеров в [Gravity UI](https://gravity-ui.com/ru/components/navigation/aside-header)
3. Поддержка Navigation в темере UIKit
4. Унификация API для subheaderItem, menuItem, footerItem

## Компоненты

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

Используется для темизирования компонентов Navigation