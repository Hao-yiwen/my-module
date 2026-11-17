# my-module

[![version](https://img.shields.io/badge/version-0.1.0-blue.svg)](./package.json)
[![license](https://img.shields.io/badge/license-MIT-green.svg)](./package.json)
[![Expo SDK](https://img.shields.io/badge/Expo%20SDK-49-000020.svg)](https://docs.expo.dev/modules/overview/)
[![React Native](https://img.shields.io/badge/React%20Native-0.72.6-61DAFB.svg)](https://reactnative.dev/)

这是一份 **Expo Module 学习示例**。仓库用 Expo Modules API 搭了一个名为 `MyModule` 的本地模块（iOS / Android / Web），并附带一个 Expo SDK 49 示例应用，页面上只显示 `hello()` 的返回值。

包名 `my-module`，版本 `0.1.0`，许可证 MIT。源码仓库：<https://github.com/Hao-yiwen/my-module>。

## 功能

JavaScript 入口是 `src/index.ts`。原生模块名都是 `MyModule`。

| 导出 | 行为 |
| --- | --- |
| `PI` | 常量。iOS 为 `Double.pi`，Android 为 `Math.PI`，Web 为 `Math.PI`。 |
| `hello()` | 同步函数。Android 与 Web 返回 `Hello world! 👋`。iOS 返回 `Hello world! 👋12312312`。 |
| `setValueAsync(value)` | 异步函数。把传入字符串通过 `onChange` 发回 JavaScript，载荷为 `{ value: string }`。 |
| `addChangeListener(listener)` | 订阅 `onChange`，返回 `expo-modules-core` 的 `Subscription`。 |
| `MyModuleView` | 视图组件，属性为 `name: string`。iOS / Android 的视图类是空的 `ExpoView`，`name` 只会打印到原生日志。Web 把 `name` 渲染成文本。 |

`example/App.tsx` 只把 `MyModule.hello()` 的结果显示在白底居中的页面上。`PI`、`setValueAsync`、事件监听和 `MyModuleView` 已经在模块里实现，示例页面没有调用它们。

```ts
import * as MyModule from 'my-module';

MyModule.hello();
console.log(MyModule.PI);

const subscription = MyModule.addChangeListener((event) => {
  console.log(event.value);
});

await MyModule.setValueAsync('demo');
subscription.remove();
```

## 技术栈与依赖

`package.json` 写的是版本范围，括号里是 `package-lock.json` 解析到的版本。

**模块（仓库根目录）**

| 依赖 | package.json | lockfile |
| --- | --- | --- |
| expo-module-scripts | ^3.0.11 | 3.1.0 |
| expo-modules-core | ^1.5.12 | 1.5.12 |
| @types/react | ^18.0.25 | 18.2.47 |

`peerDependencies` 是 `expo`、`react`、`react-native`，范围都是 `*`。示例应用把它们固定在 Expo SDK 49 这一代。

**示例应用（`example/`）**

| 依赖 | package.json | lockfile |
| --- | --- | --- |
| expo | ~49.0.15 | 49.0.21 |
| react | 18.2.0 | 18.2.0 |
| react-native | 0.72.6 | 0.72.6 |
| typescript | ^5.1.3 | 5.3.3 |
| @babel/core | ^7.20.0 | 7.23.7 |
| @types/react | ~18.2.14 | 18.2.47 |

随 Expo 安装、但未在 `example/package.json` 里单独声明的工具链包括 `babel-preset-expo@9.5.2`、`babel-plugin-module-resolver@5.0.0`。

**原生工程**

- iOS：部署目标 13.0，Swift 5.4。Pod 名 `MyModule`，依赖 `ExpoModulesCore`。示例使用 Hermes，New Architecture 未打开。
- Android：`minSdk 21`，`compileSdk` / `targetSdk 33`，Kotlin 1.8.10，Java 11，Android Gradle Plugin 7.4.2，Gradle 8.0.1。示例 `hermesEnabled=true`，`newArchEnabled=false`。应用包名 `expo.modules.mymodule.example`。

`expo-module.config.json` 的平台为 `ios`、`android`、`web`。iOS 模块类是 `MyModule`，Android 模块类是 `expo.modules.mymodule.MyModule`。Web 没有单独的原生类，由 `src/MyModule.web.ts` 和 `src/MyModuleView.web.tsx` 提供。

## 目录

```text
.
├── src/                         JavaScript / TypeScript API
│   ├── index.ts                 对外导出
│   ├── MyModule.ts              原生端 requireNativeModule('MyModule')
│   ├── MyModule.web.ts          Web 实现
│   ├── MyModule.types.ts        事件与视图的类型
│   ├── MyModuleView.tsx         原生视图封装
│   └── MyModuleView.web.tsx     Web 视图
├── ios/                         Swift 模块与 MyModule.podspec
├── android/                     Kotlin 库模块
├── example/                     Expo 示例应用（含 ios/、android/ 与图标、启动图）
├── expo-module.config.json      平台与原生模块类名
├── package.json
└── tsconfig.json                编译产物输出到 build/
```

示例在开发时直接编译 `src`，不读取 `build/`：

- `example/babel.config.js` 把包名 `my-module` 指到 `../src/index.ts`
- `example/tsconfig.json` 用 `paths` 做同样的映射
- `example/package.json` 的 `expo.autolinking.nativeModulesDir` 为 `..`，原生自动链接指向仓库根目录

`build/` 已写入 `.gitignore`。发布入口仍是 `package.json` 里的 `build/index.js` 与 `build/index.d.ts`。

## 环境要求

- 编译模块的 TypeScript：Node.js 与 npm。仓库没有 `engines` 字段。下面的核对使用 Node.js 22.23.4、npm 10.9.9。Expo SDK 49 一代通常搭配 Node.js 18。
- 运行 iOS 示例：macOS、Xcode、CocoaPods，系统不低于 iOS 13。
- 运行 Android 示例：Android SDK 33，以及与工程 `compileOptions` 一致的 JDK 11。
- `npm run open:ios` 和 `npm run open:android` 调用 macOS 的 `open`，只在 macOS 上打开 Xcode / Android Studio。

## 安装与运行

根目录和 `example/` 的 lockfile 把每个包的 `resolved` 都指向 `http://registry.npm.release.ctripcorp.com`。能访问该源时，按 lockfile 安装：

```bash
# 模块
npm ci
npm run build

# 示例
cd example
npm ci
npm start
```

`npm start` 即 `expo start`。示例目录里还有：

```bash
npm run ios       # expo run:ios
npm run android   # expo run:android
npm run web       # expo start --web
```

模块根目录的其他脚本：

| 命令 | 作用 |
| --- | --- |
| `npm run build` | `expo-module build`，输出 `build/` |
| `npm run clean` | 删除编译产物 |
| `npm run lint` | 用 ESLint 检查 `src`（`.eslintrc.js` 继承 `universe/native` 与 `universe/web`） |
| `npm test` | `expo-module test`（Jest） |
| `npm run prepare` | `expo-module prepare`，安装时会编译模块 |
| `npm run open:ios` | 用 Xcode 打开 `example/ios` |
| `npm run open:android` | 用 Android Studio 打开 `example/android` |

### 本环境核对过的命令

核对机器是 Linux，Node.js 22.23.4，npm 10.9.9，OpenJDK 21，没有 Xcode、CocoaPods，也没有 `ANDROID_HOME`。

| 命令 | 结果 |
| --- | --- |
| 根目录 `npm ci` | 失败。下载地址是 `registry.npm.release.ctripcorp.com`，本机 `ENOTFOUND`。 |
| 按 lockfile 版本从 registry.npmjs.org 安装 `expo-module-scripts@3.1.0`、`expo-modules-core@1.5.12`、`@types/react@18.2.47` 后执行 `npm run build` | 成功，生成 `build/`。 |
| 同一套依赖再装上 lockfile 中的 `eslint@8.56.0`、`prettier@3.1.1` 后执行 `npm run lint` | 退出码 0。0 个 error，18 个 warning（Prettier 的引号与换行，以及 `src/index.ts` 的 import 顺序）。 |
| `npm test` | 退出码 1。仓库里没有 `*.test.*`、`*.spec.*` 或 `__tests__`，Jest 报告 `No tests found`。 |
| 不使用内网 lockfile、按 `example/package.json` 安装后执行 `npx tsc --noEmit` | 通过。这次解析到 expo 49.0.23、react-native 0.72.6、typescript 5.9.3。lockfile 记录的是 expo 49.0.21、typescript 5.3.3。 |
| `npx expo export --platform android` | 成功，Metro 打出 Android Hermes bundle。 |
| `npm run web`（`expo start --web`） | 失败。CLI 要求安装 `react-native-web@~0.19.6`、`react-dom@18.2.0`、`@expo/webpack-config@^19.0.0`。这三项都不在 `example/package.json` 及其 lockfile 中。`example/webpack.config.js` 会加载 `@expo/webpack-config`。 |
| `expo run:ios` / `expo run:android` | 未执行。本机没有 iOS 工具链，也没有 Android SDK。 |

## 备注

- `package.json` 的 `repository`、`bugs.url`、`homepage` 写成了 `https://github.com/Hao-yiwen/-my-module`（仓库名多了一个前导连字符）。git remote 是 `https://github.com/Hao-yiwen/my-module`。
- 根目录没有单独的 `LICENSE` 文件，许可证以 `package.json` 的 `"license": "MIT"` 为准。
- `example/assets/` 里是应用图标和启动图，仓库没有界面截图。
- 原生文件中的英文注释指向 [Expo Modules API](https://docs.expo.dev/modules/module-api/)。模块总览见 [Expo Modules](https://docs.expo.dev/modules/overview/)。
