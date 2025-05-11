# 我的 Todolist PWA 開發紀錄

由 Google Gemini 生成
這份文件記錄了我們關於開發一個跨平台 Todolist Progressive Web App (PWA) 的討論過程和相關資訊。

## 1. 應用程式基本功能

一個基本的 Todolist 應用程式應包含以下功能：

* **新增任務：** 讓使用者可以輸入新的待辦事項。
* **查看任務：** 顯示目前所有的待辦事項。
* **標記完成：** 讓使用者可以將已完成的任務標記起來。
* **刪除任務：** 讓使用者可以移除不再需要的任務。

進階功能 (未來可考慮)：

* 編輯任務
* 設定截止日期/時間
* 任務分類/標籤
* 持久化儲存 (已透過 `localStorage` 實現)

## 2. 跨平台開發方案

我們討論了幾種跨平台開發方案，最終選擇從 PWA 開始：

* **網頁應用程式 (Progressive Web App - PWA)**
    * **優點：** 開發成本相對較低、易於部署和更新、觸及廣泛使用者。
    * **缺點：** 功能可能受限、瀏覽器相容性考量。
    * **適合情境：** 功能需求不複雜，希望快速開發並觸及廣大使用者。
* **跨平台框架 (React Native, Flutter, Ionic, Xamarin/.NET MAUI)**
* **其他跨平台技術 (Electron, Kivy)**

## 3. PWA 實作步驟

以下是我們討論的 PWA 實作步驟和相關程式碼：

### 3.1. HTML 結構 (index.html)

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的 Todolist</title>
    <link rel="stylesheet" href="style.css">
    <link rel="manifest" href="manifest.json">
    <link rel="apple-touch-icon" href="icon-192x192.png">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="default">
    <meta name="theme-color" content="#4CAF50">
</head>
<body>
    <div class="container">
        <h1>我的 Todolist</h1>
        <div class="input-area">
            <input type="text" id="taskInput" placeholder="新增任務">
            <button id="addTaskBtn">新增</button>
        </div>
        <ul id="taskList">
        </ul>
    </div>
    <script src="script.js"></script>
</body>
</html>
```

### 3.2. CSS 樣式表 (style.css)

```CSS
body {
    font-family: sans-serif;
    margin: 20px;
    background-color: #f4f4f4;
}

.container {
    max-width: 600px;
    margin: 0 auto;
    background-color: #fff;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

h1 {
    text-align: center;
    color: #333;
    margin-bottom: 20px;
}

.input-area {
    display: flex;
    margin-bottom: 15px;
}

#taskInput {
    flex-grow: 1;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px 0 0 4px;
    font-size: 16px;
}

#addTaskBtn {
    background-color: #4CAF50;
    color: white;
    border: none;
    padding: 10px 15px;
    border-radius: 0 4px 4px 0;
    cursor: pointer;
    font-size: 16px;
}

#addTaskBtn:hover {
    background-color: #45a049;
}

#taskList {
    list-style: none;
    padding: 0;
}

#taskList li {
    display: flex;
    align-items: center;
    padding: 10px 0;
    border-bottom: 1px solid #eee;
}

#taskList li:last-child {
    border-bottom: none;
}

#taskList li span {
    flex-grow: 1;
    font-size: 16px;
}

#taskList li button {
    background-color: #f44336;
    color: white;
    border: none;
    padding: 5px 10px;
    border-radius: 4px;
    cursor: pointer;
    margin-left: 10px;
    font-size: 14px;
}

#taskList li button:hover {
    background-color: #d32f2f;
}

#taskList li input[type="checkbox"] {
    margin-right: 10px;
}

#taskList li.completed span {
    text-decoration: line-through;
    color: #888;
}
```

### 3.3. JavaScript 檔案 (script.js)

```JavaScript
document.addEventListener('DOMContentLoaded', () => {
    const taskInput = document.getElementById('taskInput');
    const addTaskBtn = document.getElementById('addTaskBtn');
    const taskList = document.getElementById('taskList');
    let tasks = loadTasks();
    renderTasks();

    addTaskBtn.addEventListener('click', addTask);
    taskList.addEventListener('click', handleTaskAction);

    function loadTasks() {
        const storedTasks = localStorage.getItem('tasks');
        return storedTasks ? JSON.parse(storedTasks) : [];
    }

    function saveTasks() {
        localStorage.setItem('tasks', JSON.stringify(tasks));
    }

    function renderTasks() {
        taskList.innerHTML = '';
        tasks.forEach((task, index) => {
            const listItem = document.createElement('li');
            listItem.innerHTML = `
                <input type="checkbox" data-index="${index}" ${task.completed ? 'checked' : ''}>
                <span class="${task.completed ? 'completed' : ''}">${task.text}</span>
                <button data-index="${index}">刪除</button>
            `;
            taskList.appendChild(listItem);
        });
    }

    function addTask() {
        const taskText = taskInput.value.trim();
        if (taskText) {
            tasks.push({ text: taskText, completed: false });
            saveTasks();
            renderTasks();
            taskInput.value = '';
        }
    }

    function handleTaskAction(event) {
        const target = event.target;
        if (target.type === 'checkbox') {
            const index = parseInt(target.dataset.index);
            tasks[index].completed = target.checked;
            saveTasks();
            renderTasks();
        } else if (target.tagName === 'BUTTON') {
            const index = parseInt(target.dataset.index);
            tasks.splice(index, 1);
            saveTasks();
            renderTasks();
        }
    }

    // 註冊 Service Worker
    if ('serviceWorker' in navigator) {
        navigator.serviceWorker.register('/service-worker.js');
    }
});
```

### 3.4. Manifest 檔案 (manifest.json)

```JSON
{
  "name": "我的 Todolist",
  "short_name": "Todolist",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#4CAF50",
  "theme_color": "#4CAF50",
  "icons": [
    {
      "src": "icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

### 3.5. Service Worker 檔案 (service-worker.js)

```JavaScript
const cacheName = 'todolist-v1';
const staticAssets = [
    '/',
    'index.html',
    'style.css',
    'script.js',
    'manifest.json',
    'icon-192x192.png' // 確保包含你的主要圖示
];

self.addEventListener('install', async event => {
    const cache = await caches.open(cacheName);
    await cache.addAll(staticAssets);
});

self.addEventListener('activate', event => {
    event.waitUntil(caches.keys().then(keys => {
        return Promise.all(keys
            .filter(key => key !== cacheName)
            .map(key => caches.delete(key))
        );
    }));
});

self.addEventListener('fetch', event => {
    event.respondWith(caches.match(event.request)
        .then(response => {
            return response || fetch(event.request);
        })
    );
});
```
## 3.6. 應用程式圖示 (icon-144x144.png)
我們生成了一個 144x144 像素的應用程式圖示 (已在之前的回覆中提供)。你需要準備其他尺寸的圖示並在 manifest.json 中引用。

## 4. 部署到 GitHub Pages

我們討論了兩種將 PWA 部署到 GitHub Pages 的方法：

* 方法一：從主分支 (或任何其他分支) 部署
 1. 前往 GitHub 倉庫的 "Settings" -> "Pages"。
 2. 在 "Source" 中選擇你的部署分支。
 3. 點擊 "Save"。
 4. 等待部署完成並查看部署網址。

* 方法二：從 gh-pages 分支部署
 1. 建立並切換到 gh-pages 分支。
 2. 將 PWA 檔案複製到 gh-pages 分支的根目錄並推送。
 3. 在 GitHub 倉庫的 "Settings" -> "Pages" 中選擇 "gh-pages branch" 作為部署來源。
 4. 點擊 "Save"。
 5. 等待部署完成並查看部署網址。

注意事項：

* 確保 Service Worker 的作用域和路徑正確。
* GitHub Pages 預設使用 HTTPS，這是 Service Worker 運作的必要條件。

## 5. 後續步驟

測試你的 PWA 在不同平台和瀏覽器上的表現。
逐步添加更多進階功能。
持續優化使用者體驗。

希望這個紀錄對你有幫助！如果你有任何其他問題，隨時可以再問我。祝你的 Todolist 應用程式開發順利！