# Cách build ReactJS sang Electron cho hệ điều hành Window, MacOS, Linux.
* Sử dụng Node version: `v14.18.1`
* Đã thử trên Node `v17` đã có lỗi !

## Cài đặt chung
1. Tạo ứng dụng mới của chúng tôi bằng Tạo ứng dụng React.
```
npx create-react-app my-app
cd my-app
```
2. Thêm Electron vào trong React.
```
yarn add electron electron-builder --dev
```
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
- Tập lệnh này sẽ chỉ đợi cho đến khi CRA chạy ứng dụng React trên localhost: 3000 trước khi khởi động Electron.
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
```
yarn electron-dev
```

# Build thành App
1. Chạy dòng lệnh để cài đặt Bản ghi lại:
```
yarn add @rescripts/cli @rescripts/rescript-env --dev
```
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
```js
module.exports = [require.resolve('./.webpack.config.js')]
```
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
```
yarn add electron-builder typescript --dev
```
2. Đặt thuộc tính **"homepage"** vào **package.json**
```
"homepage": "./",
```
- CRA, theo mặc định, xây dựng một index.html sử dụng các đường dẫn tuyệt đối.
- Điều này sẽ không thành công khi tải nó trong Electron
- Nên ta thiết lập homepage để thay đổi.
3. Thêm phần sau vào thẻ **"scripts"** trong **package.json**.
```json
"postinstall": "electron-builder install-app-deps",
"preelectron-pack": "yarn build",
"electron-pack": "electron-builder -mwl"
```
 - `"postinstall": "electron-builder install-app-deps"` sẽ đảm bảo rằng các phụ thuộc gốc luôn khớp với phiên bản electron.
 - `"preelectron-pack": "yarn build"` sẽ xây dựng CRA.
 - `"electron-pack": "build -mwl"` gói ứng dụng cho Mac (m), Windows (w), Linux (l).
 * Muốn chỉ build riêng hệ điều hành nào thì chỉ ghi ký hiệu hệ điều hành đó trong `electron-pack`.
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
```
yarn electron-pack
```
- File app sẽ trong thư mục dist.

# Giảm kích thước ứng dụng Electron - ReactJs
- Di chuyển hết các gói package ở `dependencies` sang `devDependencies` trong `package.json`. Chỉ giữ lại gói `electron-is-dev`.
- Xoá bỏ `node_modules`
- Chạy lại lệnh dưới để cập nhật lại node_modules
```
yarn
```

# Tạo icon app Electron.
* Để tạo icon cho App Electron, ta nên dùng ảnh `.PNG` có kích thước phải là 512x512.
 - Nên dùng ảnh `.PNG` là gì file ảnh `.PNG` hỗ trợ được cả 3 nền tảng MacOS, Window, Linux.
 - Kích thước phải là 512x512 là vì MacOS cần tối thiểu là 512x512. Với Window và Linux tối thiểu là 256x256. Vậy nên muốn dùng cho cả đa hệ điều hành thì cần ảnh 512x512.
 - Nếu sai thì electront tự động dùng icon mặc định.
* Các bước làm:
- Đầu tiên ta đưa ảnh vào thư mục `build`. Riêng Linux thì chuyển vào mục `build/icon`.
- Ở trong thẻ `build` ở `package.json`. Ta thêm thẻ:
```json
"mac": {
    "icon": "./public/icons/mac/icon.icns",     <---------- set Mac Icons
    "category": "public.app-category.utilities"
  },
  "win": {
    "icon": "./public/icons/png/256x256.png" <---------- set Win Icon
  },
```
# Thêm Notification vào trong Electron
* Khi ta muốn bắn 1 Notification, ta sẽ bắt sự kiện chạy vào hàm như sau:
```js
const NOTIFICATION_TITLE = 'Title' // Tiêu đề của Notification
const NOTIFICATION_BODY = 'Notification from the Renderer process. Click to log to console.' //Nội dung của Notification

new Notification(NOTIFICATION_TITLE, { body: NOTIFICATION_BODY })
  .onclick = function (event) {
  // Khi click vào Notification sẽ chạy các lệnh trong này.
  };
```
* Tạo icon cho Notification
> Tham khảo: https://www.electronjs.org/docs/latest/api/notification
- Ta viết vào tham số thứ 2 của Notification.
```js
//V Ví dụ
new Notification(NOTIFICATION_TITLE, {
  body: NOTIFICATION_BODY,
  icon: ICON_NOTIFICATION, })
```
- Để tránh bị lỗi ta nên import ICON_NOTIFICATION ở ngoài function components. Ví dụ:
```js
import ICON_NOTIFICATION from './assets/test.png'
```
* Hoặc có cách viết khác như sau: 
```js
const newNotification = new Notification(NOTIFICATION_TITLE, { body: NOTIFICATION_BODY });

newNotification .onclick = function (event) {
    event.preventDefault(); // prevent the browser from focusing the Notification's tab
    window.open("google.com","_blank");
};
```
# Các lỗi thường gặp và các khắc phục
1.
* Lỗi: 
```
$ concurrently "BROWSER=none yarn start" "wait-on http://localhost:3000 && electron ."
[0] 'BROWSER' is not recognized as an internal or external command,
[0] operable program or batch file.
[0] BROWSER=none yarn start exited with code 1
```
* Sữa lỗi: 
- Chạy lệnh sau: `yarn add cross-env`
- Thêm mới một lệnh có thể bất kỳ (ví dụ đặt tên là `dev` chẳng hạn) hoặc sửa lệnh `electron-dev` trong thẻ `scripts` ở file `package.json` thành
* Thêm mới 1 thẻ:
```json
"dev": "concurrently \"cross-env BROWSER=none yarn start\" \"wait-on http://localhost:3000 && electron .\""
```
* Hoặc sửa thẻ `electron-dev` cũ thành
```json
"electron--dev": "concurrently \"cross-env BROWSER=none yarn start\" \"wait-on http://localhost:3000 && electron .\""
```
> Sau đó chạy lệnh `yarn dev` hoặc `yarn electron-dev` tương ứng.

3.
* Lỗi: 
```
Package "electron" is only allowed in "devDependencies". Please remove it from the "dependencies" section in your package.json.
Package "electron-builder" is only allowed in "devDependencies". Please remove it from the "dependencies" section in your package.json.
error Command failed with exit code 1.
```
* Sửa lỗi:
> Xóa 2 thuộc tính `"electron"`, `"electron-builder"`, trong mục `"dependencies"`
4.
* Lỗi:
>'react-scripts' is not recognized as an internal or external command, operable program or batch file.
* Sửa lỗi:
Chạy lệnh: ``` yarn add react-scripts --dev ```
5.
* Lỗi:
>'electron-builder' is not recognized as an internal or external command, operable program or batch file.
* Sửa lỗi:
> yarn add electron-builder --dev
# Các vấn đề khác
1. Nếu file App.tsx chỉ viết router thì Reactjs và bản build Electronjs vẫn ổn. Nhưng build ra file .exe vì nó lấy mặc định màn hình chính là App.tsx, nên là khi build ra .exe thì app ko hiện gì cả.
> Cách sửa lỗi: --> Đổi BrowserRouter thành HashRouter.
2. Các file icon .svg không hiện thị được trong Electron.
- Thêm code này vào function createWindow của file electron.js:
```js
 win.webContents.on('dom-ready', () => {
            fs.readFile(path.join(__dirname, 'logo192.png'), 'utf8', (err, data) => {
                if (err) throw err
                    win.webContents.executeJavaScript(`
                        var doc = new DOMParser().parseFromString(
                            '${data}',
                            'application/xml')
                        var svgHolder = document.getElementById('svgtest') // is just a <div>
                    svgHolder.appendChild(svgHolder.ownerDocument.importNode(doc.documentElement, true))
                `)
            })
        })
  ```
  # Tạo các sự khác biệt trên từng hệ điều hành
* Sử dụng package: `react-device-detect`
* Link: https://www.npmjs.com/package/react-device-detect
* Các thứ liên quan: 
- https://bestofreactjs.com/repo/duskload-react-device-detect-react-utilites
- https://www.npmjs.com/package/classnames
* Đang tìm hiểu cơ bản về thay đổi css khi ở các hệ điều hành khác nhau.
 - Đầu tiên import package `react-device-detect` và package `classnames`
 ```
 yarn add react-device-detect
 yarn add classnames
 ```
 ở code thì làm ví dụ đơn giản như sau:
 - File App.tsx
```txs
import React from 'react';
import classNames from 'classnames';
import {isWindows, isMacOs} from 'react-device-detect'
import './App.css';

function App() {
  return (
    <div className={classNames('App', {
      'isMacOs': isMacOs,
      'isWindows': isWindows
    })}>
      <h1>Test</h1>
    </div>
  );
}

export default App;
```
- Ở file App.css
```
.App {
  background: red;
}
.App.isMacOs {
  background:green;
}
.App.isWindows {
  background:yellow;
}
```
Khi ở từng hệ điều hành thì sẽ hiện thị màu khác nhau. Mặc định chung là màu `đỏ`.
