## 0. Electron介绍

[Electron](https://electron.atom.io/)是基于HTML、CSS、JavaScript的**桌面开发**的解决方案，跨平台。用过的软件里VSCode、Atom等是用Electron开发的。



## 1. 快速入门

参照以下例子就能运行一个入门级的Electron demo

```
# 克隆这仓库
$ git clone https://github.com/electron/electron-quick-start
# 进入仓库
$ cd electron-quick-start
# 安装依赖库并运行应用
$ npm install && npm start
```



文档目录如下，且只需要一个`electron`依赖包，

```
your-app/
├── package.json
├── main.js
└── index.html
```



electron的学习资源，如下所列，

可以下载并运行[Electron API Demos app](https://github.com/electron/electron-api-demos) ，用来查看 Electron 的一些API。

详细中文文档可参阅[Electron 中文文档](https://www.w3cschool.cn/electronmanual/) 。

其它资源，

[electron内部结构](https://github.com/ilyavorobiev/atom-docs/blob/master/atom-shell/Architecture.md)

[awesome-electron](https://github.com/sindresorhus/awesome-electron)

[electron-sample-apps](https://github.com/hokein/electron-sample-apps)



##  2. 主要概念

### 主进程

**主进程**运行于 `package.json` 里 `main` 脚本，用来创建 GUI。



### 渲染进程

每个页面都在运行着一个**渲染进程**，每个渲染进程只关心自己的网页。

**渲染进程**若是想访问GUI操作，需要向**主进程**发出请求，两种进程之间的交互方式有：通过`ipcRenderer`和`ipcMain`来发送messages，通过 [remote](https://electron.org.cn/doc/api/remote.html) 模块进行RPC 方式通讯 。



### 主进程与渲染进程的区别

主进程使用 `BrowserWindow` 实例创建页面。每个 `BrowserWindow` 实例都在自己的渲染进程里运行页面。当一个 `BrowserWindow` 实例被销毁后，相应的渲染进程也会被终止。

主进程管理所有页面和与之对应的渲染进程。

Electron提供一些内置组件用于开发，一些组件只可以在主进程中使用，一些只可以在渲染进程中使用，但是也有部分可以在这两种进程中都可使用。

由于在页面里管理原生 GUI 资源非常危险且容易造成资源泄露，所以在页面调用 GUI 相关的 APIs 是不被允许的。若想在网页里使用 GUI 操作，其对应的渲染进程必须与主进程进行通讯，请求主进程进行相关的 GUI 操作。

**基本规则**：GUI模块或系统底层模块只可在**主进程**中使用。





## 3. 打包发布

采用第三方打包工具`electron-builder`，可以生成多种target formats:

- All platforms: `7z`, `zip`, `tar.xz`, `tar.lz`, `tar.gz`, `tar.bz2`, `dir` (unpacked directory).
- [macOS](https://electron.build/configuration/configuration#MacOptions-target): `dmg`, `pkg`, `mas`.
- [Linux](https://electron.build/configuration/configuration#LinuxBuildOptions-target):  [AppImage](http://appimage.org/),  [snap](http://snapcraft.io/), debian package (`deb`), `rpm`, `freebsd`, `pacman`, `p5p`, `apk`.
- [Windows](https://electron.build/configuration/configuration#WinBuildOptions-target): `nsis` (Installer), `nsis-web` (Web installer), `portable` (portable app without installation), AppX (Windows Store), Squirrel.Windows

```
$ yarn add electron-builder --dev
```

使用yarn安装为devDependencies，否则可能报错。

关于script的配置如下，

```
"scripts": {
 	"start": "electron .",
    "pack:mac": "ELECTRON_MIRROR=http://npm.taobao.org/mirrors/electron/ electron-builder --mac",
    "pack:win": "ELECTRON_MIRROR=http://npm.taobao.org/mirrors/electron/ electron-builder --win --ia32"
 },
 "build": {
  	"appId": "your.id",
    "mac": {
      "category": "your.app.category.type"
    }
 }
```

其中，设置环境变量 ELECTRON_MIRROR，采用淘宝electron [镜像](http://npm.taobao.org/mirrors) ，可以大大提速。

```
$ yarn run pack:mac
```

执行以上命令之后，生成`/dist`目录，存放相应的target。









