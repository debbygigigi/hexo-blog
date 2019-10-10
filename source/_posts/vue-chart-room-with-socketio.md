---
title: vue + socket.io = chatroom
date: 2018-08-29 21:29:43
tags:
---

![](https://i.imgur.com/fnWR7hx.png)

## 前言

因為看了吳大師的 [[週四寫程式系列] 來用socket.io做個即時互動的遊戲吧！](https://www.youtube.com/watch?v=QVZcMx3jtG8) Youtube直播，對這個主題很有興趣很想來挑戰
本來想說應該不會太難，老師直播才用了一小時，那我兩小時應該可以吧～
結果雖然頁面做得很快，但想要deploy到heroku上debug就花了將近一個禮拜的晚上（哭）

## 準備

使用 [vue-cli](https://github.com/vuejs-templates/webpack) 的template建立新專案

```
vue init webpack-simple vue-chatroom
```

安裝套件

```
npm install express socket.io --save
```

### server端

首先在專案的根目錄下建一個跑後端的檔案 `server.js` 
這個server.js主要要建立 `socket server`，並幫我們導向靜態頁面

*server.js* 
```javascript=
const express = require('express');
const socketIO = require('socket.io');
const path = require('path');
const serveStatic = require('serve-static');

const PORT = process.env.PORT || 3000;

// 建立http server 
const server = express()
  .use("/", serveStatic(path.join(__dirname, '/dist')))
  .listen(PORT, () => console.log(`Listening on ${ PORT }`));

// 建立socket server 
const io = socketIO(server);

// 處理socket連線事件
io.on('connection', (socket) => {
  console.log('Client connected');
  socket.on('disconnect', () => console.log('Client disconnected'));
});
```

因為我的 `index.html`位置在`/dist`下，透過 `serveStatic` 這個套件幫我導向

### client端

安裝套件 [socket.io-client](https://github.com/socketio/socket.io-client)
```
npm i socket.io-client
```

*app.vue*
```javascript=
import io from "socket.io-client";
var socket = io();
```

如果你的socket server跟client是分開的
`io()` 括號內可以填入socket server的網址
如果沒填就是預設連到目前的網址

### 測試連線

server端跟client端的基本設置就完成了
現在只要開啟server就能知道有無連線成功

```
node server.js
```

如果terminal有出現 `Client connected` 就是連線成功！

### 測試接收訊息

在client端連線後，`emit`訊息到server

```
var socket = io();
socket.emit("message", "hello world.");
```

server端用`on`接收，後面接callback function
```
io.on('connection', function (socket) {
    console.log('a user connected.');

    socket.on('message', (msg) => {
        console.log('user says: ' + msg);
    });
});
```

terminal出現訊息就代表server接收成功！

## 實作功能

### 顯示訊息

首先建立 `messages` 陣列來儲存訊息
假設一開始已經有一筆訊息，在一開始連線時就要讓client收到這個歷史訊息

server端 

```javascript=
// 建一個儲存歷史訊息的變數，只要server開著，變數內容就會一直存在
const messages = [
    { name: 'Majar', message: 'Good Night.' }
];

io.on('connection', function (socket) {
    console.log('a user connected.');
    // 廣播 歷史訊息
    socket.emit('syncMessages', messages);
});
```

client端

先建立簡單的list畫面

*App.vue*
```htmlmixed=
<template>
  <div id="app">
    <h1>聊起來</h1>
    <ul>
      <li v-for="msg in messages">
        {{ msg.name }}: 
        {{ msg.message }}
      </li>
    </ul>
  </div>
</template>
```

收到 `syncMessages` 事件，把訊息資料傳到data去render

```javascript=
//...
data() {
    return {
      messages: []
    };
  },
created() {
    socket.on("syncMessages", messages => {
      this.messages = messages;
    });
  }
```

### client留言

接下來就是要讓使用者可以留言

client端
寫個簡單的input和button

*App.vue*
```htmlmixed=
<input ref="name" type="text" placeholder="name" value="路人">
<input ref="message" type="text" placeholder="message">
<button @click="addMessage">留言</button>
```

按下新增留言按鈕時，emit `newMessage` 事件，並帶上資料

```javascript=
methods: {
    addMessage() {
      socket.emit("newMessage", {
        name: this.$refs.name.value,
        message: this.$refs.message.value
      });
      this.$refs.name.value = "";
      this.$refs.message.value = "";
    }
}
```


server端

當收到client的訊息時，push至message陣列後
並再次廣播到client，確認每個client都有更新到最新訊息

```javascript=
// 接收新訊息
socket.on('newMessage', (msg) => {
    // console.log(msg.name + ' says: ' + msg.message);
    messages.push(msg);
    socket.emit('syncMessages', messages);
});

```

以上基本的的功能就完成了

## 遇到的問題

在deploy的過程中一直遇到問題而且卡關卡很久
最困難的是，還是不知道問題出在哪

![](https://i.imgur.com/XsPp7Wp.png)

![](https://i.imgur.com/V2xxWis.png)

第一個error讓我以為ㄧ定要有HTTPS，害我還去申請 [openSSL](https://www.openssl.org/) 而且還看不是很懂...
後來看到官方的這篇 [Using WebSockets on Heroku with Node.js](https://devcenter.heroku.com/articles/node-websockets)，我就跟著它做然後就成功了

原來是因為在看吳老師的影片時，以為ㄧ定要有 `socket server` 和 `api server`，而且我把前端路由做在`api server`上，所以一直出錯
後來跟著官方教學用一個`socket server`並把前端路由做在這邊就可以了
我覺得也是因為我對後端的觀念比較不熟才會一直卡關，甚至連google都沒用，因為不知道實際的問題到底在哪
早知道有官方教學就能少走好多天的冤枉路啊


## 後記

本來想說先做簡單的功能，deploy上線之後在慢慢增加功能、調整版面
真的沒想到deploy花了好多時間（哭）

來小小的寫一下features，希望之後有時間可以完成更多功能！

* [ ] firebase儲存訊息
* [ ] 有人輸入中
* [ ] 有幾個人在線上