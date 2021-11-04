# Cách build ReactJS sang Electron cho hệ điều hành Window, MacOS, Linux.

## Cài đặt chung
1. Tạo ứng dụng mới của chúng tôi bằng Tạo ứng dụng React.
```
npx create-react-app my-app
cd my-app
```
2. Thêm Electron vào trong React.
> yarn add electron electron-builder --dev
3. Thêm một số công cụ sẽ cần.
```
yarn add wait-on concurrently --dev
yarn add electron-is-dev
```

## Chỉnh sửa code
1. Tạo một tệp mới public/electron.js, với các nội dung sau:
```js
const electron = require('electron');
const app = electron.app;
const BrowserWindow = electron.BrowserWindow;

const path = require('path');
const isDev = require('electron-is-dev');

let mainWindow;

function createWindow() {
  mainWindow = new BrowserWindow({width: 900, height: 680});
  mainWindow.loadURL(isDev ? 'http://localhost:3000' : `file://${path.join(__dirname, '../build/index.html')}`);
  if (isDev) {
    // Open the DevTools.
    //BrowserWindow.addDevToolsExtension('<location to your react chrome extension>');
    mainWindow.webContents.openDevTools();
  }
  mainWindow.on('closed', () => mainWindow = null);
}

app.on('ready', createWindow);

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});

app.on('activate', () => {
  if (mainWindow === null) {
    createWindow();
  }
});
```
2. Thêm lệnh sau vào thẻ "scripts" trong package.json:
```json
"electron-dev": "concurrently \"BROWSER=none yarn start\" \"wait-on http://localhost:3000 && electron .\""
```
Tập lệnh này sẽ chỉ đợi cho đến khi CRA chạy ứng dụng React trên localhost: 3000 trước khi khởi động Electron.
3. Thêm thẻ "main" sau vào package.json:
```json
"main": "public/electron.js",
```
***
Bây giờ package.json sẽ như sau:
```json
{
  "name": "my-app",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "electron-is-dev": "^1.0.1",
    "react": "^16.8.3",
    "react-dom": "^16.8.3",
    "react-scripts": "2.1.5"
  },
  "main": "public/electron.js",
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "electron-dev": "concurrently \"BROWSER=none yarn start\" \"wait-on http://localhost:3000 && electron .\""
  },
  "eslintConfig": {
    "extends": "react-app"
  },
  "browserslist": [
    ">0.2%",
    "not dead",
    "not ie <= 11",
    "not op_mini all"
  ],
  "devDependencies": {
    "concurrently": "^4.1.0",
    "electron": "^4.0.6",
    "electron-builder": "^20.38.5",
    "wait-on": "^3.2.0"
  }
}
```
***
Có thể chạy ứng dụng ở chế độ phát triển thông qua:
> yarn electron-dev

## Build thành App
1. Chạy dòng lệnh để cài đặt Bản ghi lại:
> yarn add @rescripts/cli @rescripts/rescript-env --dev
2. Thay đổi các thẻ "scripts" ở package.json từ
```json
"start": "react-scripts start",
"build": "react-scripts build",
"test": "react-scripts test",
```
thành như sau:
```json
"start": "rescripts start",
"build": "rescripts build",
"test": "rescripts test",
```
3. Thêm 1 tệp mới ở project có tên **.rescriptsrc.js** với nội dung sau:
```module.exports = [require.resolve('./.webpack.config.js')]```
4. Thêm 1 tệp mới ở project có tên **.webpack.config.js** với nội dung sau:
```js
// define child rescript
module.exports = config => {
  config.target = 'electron-renderer';
  return config;
}
```
## Thiết lập
1. Thêm Electron Builder & Typescript:
> yarn add electron-builder typescript --dev
2. Đặt thuộc tính **"homepage"** vào **package.json**
> "homepage": "./",
- CRA, theo mặc định, xây dựng một index.html sử dụng các đường dẫn tuyệt đối.
- Điều này sẽ không thành công khi tải nó trong Electron
- Nên ta thiết lập homepage để thay đổi.
3. Thêm phần sau vào thẻ **"scripts"** trong **package.json**.
```json
"postinstall": "electron-builder install-app-deps",
"preelectron-pack": "yarn build",
"electron-pack": "electron-builder -mw"
```
 - `"postinstall": "electron-builder install-app-deps"` sẽ đảm bảo rằng các phụ thuộc gốc luôn khớp với phiên bản electron.
 - `"preelectron-pack": "yarn build"` sẽ xây dựng CRA.
 - `"electron-pack": "build -mwl"` gói ứng dụng cho Mac (m) và Windows (w).
 * Mình chỉ chọn 1 trong **electron-pack**
 4. Thêm phần sau vào **package.json**:
 ```json
 "author": {
  "name": "Your Name",
  "email": "your.email@domain.com",
  "url": "https://your-website.com"
},
"build": {
  "appId": "com.my-website.my-app",
  "productName": "MyApp",
  "copyright": "Copyright © 2019 ${author}",
  "mac": {
    "category": "public.app-category.utilities"
  },
  "files": [
    "build/**/*",
    "node_modules/**/*"
  ],
  "directories": {
    "buildResources": "assets"
  }
}
```
* Các value trong cặp key: value của thẻ author và build có thể thay đổi được.
5. Chạy lệnh này để đóng gói ứng dụng.
> yarn electron-pack

### File app sẽ trong thư mục dist.
