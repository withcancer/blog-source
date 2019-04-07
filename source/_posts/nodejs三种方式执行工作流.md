---
title: nodejs中三种方式执行任务流
date: 2016-4-29
categories:
- 前端
- js
- ES6
tags:
- 前端
- javascript
- ES6
- Promise
- async/await
---
nodejs作为异步平台，有时候也有按顺序执行同步任务的需求。假设有这样一个任务：

首先根据singer去数据库中取ablum,然后从ablum中取song。看以看出，这些任务都是后面依赖前面的，所以传统的异步方法是不行的。有以下三种方法来完成这个任务：

## Async.parallel
使用``async``库中的方法来完成这个任务：
``` javascript
function parallelDemo(req, res, next) {
    async.parallel({
        albums: cb => {
            restApi.getAblums(singer).then((response) => {
                if (response.statusCode !== 200) {
                    return cb('Unable to process ablum request');
                }
                return cb(null, albums);
            })
        },
        songs: cb => {
            restApi.getSongs(album).then((response) => {
                if (response.statusCode !== 200) {
                    return cb('Unable to process song request');
                }
                return cb(null, songs);
            })
        }
    }, (err, result) => {
        if (err) {
            return res.status(500).send(err);
        }
        return res.json(result);
    })
}
```

## 使用Promise.all
``` javascript
function parallelDemo(req, res, next) {
    Promise.all([
            getAblums(singer),
            getSongs(ablum)
        ])
        .then(results => {
            let [albums, songs] = results;
            return res.json({
                albums: albums.data,
                songs: songs.data
            });
        })
        .catch(err => {
            return res.status(500).send(err);
        })
}
```

## 使用async,await
``` javascript
 const processRequest = async () => {
        const [posts, albums, photos] = await Promise.all([
            getAblums(singer),
            getSongs(ablum)
        ]);
        return {
            albums: albums.data,
            songs: songs.data
        };
    };
processRequest()
    .then(rsp => {
        return res.json(rsp);
    })
    .catch(err => {
        return res.stats(500).send(err);
    });
```