---
title: "Expressjs"
date:  2023-06-25 14:50:52 +0800
categories: [api]
tags: [nodejs, expressjs]
---






##  Router

expressjs router 可以设计为**模块**，挂载在 app 上，它也被称之为[mini-app](https://expressjs.com/en/guide/routing.html)

router 定义 route

```js
router.get('/', (req, res) => {
  res.send('Birds home page')
})
```

router 挂载 app

```js
app.use('/birds', birds)
```
