# 自定义Electron.exe的使用


初始化环境可以阅读[这篇](/posts/build_electron/)。

上接之前的构建过程，经过第一次构建之后，后续修改源码添加 api 等都不会再构建那么久了。

## 添加简单的 API

当前目录在构建时相同的目录下，其下包含`src`、`.gclient`、`node_modules`等。

找到`src/electron`目录，在`lib/browser/api/app.ts`添加：

```bash
app.getElectronVersion = () => { // This is for test
  return 'dodola test demo' + process.versions.electron;
};
```

再到`electron.d.ts`中更新添加：

```bash
declare namespace Electron {
  const NodeEventEmitter: typeof import('events').EventEmitter;

  type Accelerator = string;
  type Event<Params extends object = {}> = {
    preventDefault: () => void;
    readonly defaultPrevented: boolean;
  } & Params;

  interface App extends NodeJS.EventEmitter {
    getElectronVersion(): string; // This is for test
...
```

然后在命令行重新构建一遍：

```bash
e build
INFO Auto-updates disabled - skipping update check
Running "autoninja.bat -j 200 electron" in D:\playground\Projects\electron-coins\electron\src\out\Testing
Proxy started successfully.
[4/4] LINK electron.exe electron.exe.pdb
RBE Stats: down 0 B, up 0 B, 1 local fallback
```

这样就准备好了测试用的自定义 electron 啦

## 测试 demo

找到一个测试用的小 electron demo，可以参考[Electron 教程](https://www.electronjs.org/zh/docs/latest/tutorial/tutorial-first-app)。

### 修改启动命令

在`package.json`中修改启动脚本`start`的部分，原来是`electron .`启动，将`electron`改为构建的`electron.exe`路径：

```bash
{
  "name": "electron-demo",
  "version": "1.0.0",
  "main": "main.js",
  "scripts": {
    "start": "chcp 65001 &&  ..\\electron\\src\\out\\Testing\\electron.exe .",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "dodola",
  "license": "MIT",
  "devDependencies": {
    "electron": "^35.2.0"
  }
}
```

{{< admonition type=info title="不重要的小说明" open=false >}}
`chcp 65001`只是为了在命令行中能够正常显示中文字符用的
{{< /admonition >}}

### 调用`getElectronVersion()`

在`main.js`中添加 console.log 语句：

```js
console.log(`Custom Electron version: ${app.getElectronVersion()}`)
```

之后正常启动测试的项目即可：

```bash
npm start
```

可以看到命令行对应的输出：

![image.png](https://img.dodolalorc.cn/i/2025/04/27/680df4e196674.png)

## 小 demo 的代码

### `main.js`

```js
const { app, BrowserWindow, ipcMain } = require('electron/main')
const path = require('node:path')
const { dialog } = require('electron/main')

let mainWindow;
const createWindow = () => {
  mainWindow = new BrowserWindow({
    width: 1000,
    height: 800,
    title: 'Just a little demo',
    icon: path.join(__dirname, 'icon.png'),
    webPreferences: {
      contextIsolation: true,
      preload: path.join(__dirname, 'preload.js')
    }
  })
  mainWindow.setMenu(null)
  mainWindow.loadFile('index.html')

  let wc = mainWindow.webContents;

  // wc.openDevTools();

  wc.on('dom-ready', (e) => {
    dialog.showMessageBox(options = {
      title: 'Hello',
      message: 'This is a message box',
    }).then((result) => {
      console.log(result);
    });
  });
}


app.whenReady().then(() => {
  ipcMain.handle('ping', () => 'pong');

  createWindow();
  mainWindow.maximize();

  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit()
});

console.log(`Custom Electron version: ${app.getElectronVersion()}`)
```

### `preload.js`

```js
const { contextBridge, ipcRenderer } = require('electron/renderer')

contextBridge.exposeInMainWorld('versions', {
  node: () => process.versions.node,
  chrome: () => process.versions.chrome,
  electron: () => process.versions.electron,
  ping: () => ipcRenderer.invoke('ping')
})
```

### `renderer.js`

```js
const information = document.getElementById('info')
information.innerText = `This app is using Chrome (v${window.versions.chrome()}), Node.js (v${window.versions.node()}), and Electron (v${window.versions.electron()})`;

const pingButton = document.getElementById('ping');
const pingMsg = document.getElementById('ping-msg');
pingButton.addEventListener('click', () => {
  window.versions.ping().then((response) => {
    console.log(response) // prints out 'pong'
    pingMsg.innerText = response
  })
})
```

### `index.html`

```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8" />
  <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'" />
  <meta http-equiv="X-Content-Security-Policy" content="default-src 'self'; script-src 'self'" />
</head>

<body>
  <h1>Hello from Electron renderer!</h1>
  <p>👋</p>
  <p id="info"></p>
  <button id="ping">ping</button>
  <p>Message from ping is: <span id="ping-msg"></span></p>
</body>
<script src="./renderer.js"></script>

</html>
```

