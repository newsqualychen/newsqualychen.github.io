---
title: 安装解锁网易云（缓存代理版）+ Music Assistant」
featured: true
id: 01a0f946-9523-73ff-af24-e9c6c553ed26
date: 2026-10-02 05:02:36
auther: squaly
featured_image: "./featured-image.jpg"
summary: 目标： MA 使用网易云音乐源 支持扫码登录 支持搜索 支持播放解灰 播放 URL 缓存，加快第二次播放 不污染 Ubuntu 系统 最终结构： Music Assistant        |        |        ↓ncm-adapter 2001(缓存代理层)     
permalink: /archives/XcU7szXb
categories:
 - default
tags: 
 - navidrome
---


* 目标：
  * MA 使用网易云音乐源
  * 支持扫码登录
  * 支持搜索
  * 支持播放解灰
  * 播放 URL 缓存，加快第二次播放
  * 不污染 Ubuntu 系统

最终结构：

```
Music Assistant
        |
        |
        ↓
ncm-adapter :2001
(缓存代理层)
        |
        |
        ↓
ncm-api :2000
(NetEaseCloudMusicApi)
        |
        |
        ↓
网易云音乐
```

---

# 一、安装 ncm-api-enhanced

## 1. 创建目录

```
mkdir -p ~/docker/ncm-api
cd ~/docker/ncm-api
```

---

## 2. 创建 compose

```
nano docker-compose.yml
```

内容：

```
services:

  ncm-api:
    image: moefurina/ncm-api:latest
    container_name: ncm-api
    restart: unless-stopped

    ports:
      - "2000:3000"

    volumes:
      - ./data:/app/data
```

保存。

---

## 3. 启动

```
docker compose up -d
```

检查：

```
docker ps | grep ncm
```

应该看到：

```
ncm-api
0.0.0.0:2000->3000
```

---

# 二、测试 ncm-api

## 1. 测试二维码接口

```
curl http://localhost:2000/login/qr/key
```

返回：

```
{
 "data":{
   "code":200,
   "unikey":"xxxx"
 }
}
```

说明正常。

---

## 2. 测试搜索

```
curl "http://localhost:2000/search?keywords=%E5%91%A8%E6%9D%B0%E4%BC%A6"
```

---

## 3. 测试播放地址

```
curl \
"http://localhost:2000/song/url/match?id=400164559"
```

返回：

```
{
"code":200,
"data":"http://xxx.mp3"
}
```

到这里：

✅ ncm-api 完成

---

# 三、安装 ncm-adapter 缓存代理

作用：

解决：

* MA 不兼容部分接口
* 播放第一次慢
* 重复请求网易云

---

## 1. 创建目录

```
mkdir -p ~/docker/ncm-adapter
cd ~/docker/ncm-adapter
```

---

# 四、创建 Dockerfile

```
nano Dockerfile
```

内容：

```
FROM python:3.12-slim


WORKDIR /app


COPY app.py .


RUN pip install --no-cache-dir \
    fastapi \
    uvicorn \
    httpx


CMD [
"uvicorn",
"app:app",
"--host",
"0.0.0.0",
"--port",
"3000"
]
```

注意：

这里安装的是容器内部 Python 包。

不会影响 Ubuntu。

---

# 五、创建 app.py

```
nano app.py
```

内容：

```
from fastapi import FastAPI, Request
from fastapi.responses import Response
import httpx
import sqlite3
import time


app = FastAPI()


NCM_API="http://ncm-api:3000"


DB="/data/cache.db"

CACHE_TIME=86400



def init_db():

    conn=sqlite3.connect(DB)

    c=conn.cursor()

    c.execute("""
    CREATE TABLE IF NOT EXISTS cache
    (
        id INTEGER PRIMARY KEY,
        url TEXT,
        created INTEGER
    )
    """)

    conn.commit()
    conn.close()



def get_cache(song_id):

    conn=sqlite3.connect(DB)

    c=conn.cursor()

    c.execute(
        "select url,created from cache where id=?",
        (song_id,)
    )

    row=c.fetchone()

    conn.close()


    if not row:
        return None


    if time.time()-row[1] < CACHE_TIME:

        print(
            "CACHE HIT",
            song_id
        )

        return row[0]


    return None




def set_cache(song_id,url):

    conn=sqlite3.connect(DB)

    c=conn.cursor()


    c.execute(
        """
        insert or replace into cache
        values(?,?,?)
        """,
        (
            song_id,
            url,
            int(time.time())
        )
    )


    conn.commit()

    conn.close()



@app.on_event("startup")
def startup():

    init_db()



@app.api_route(
"/song/url/v1",
methods=["GET"]
)
async def song_url(request:Request):


    sid=request.query_params.get("id")


    if sid:

        cache=get_cache(int(sid))


        if cache:

            return {
                "code":200,
                "data":[{
                    "id":int(sid),
                    "url":cache,
                    "br":320000,
                    "type":"mp3",
                    "level":"exhigh"
                }]
            }



    async with httpx.AsyncClient(
        timeout=60
    ) as client:


        r=await client.get(
            f"{NCM_API}/song/url/v1",
            params=request.query_params
        )


    data=r.json()


    try:

        set_cache(
            int(sid),
            data["data"][0]["url"]
        )

    except:

        pass


    return data





@app.api_route(
"/{path:path}",
methods=[
"GET",
"POST"
]
)
async def proxy(path:str,request:Request):


    async with httpx.AsyncClient(
        timeout=60
    ) as client:


        r=await client.request(
            request.method,
            f"{NCM_API}/{path}",
            params=request.query_params,
            content=await request.body()
        )


    return Response(
        content=r.content,
        status_code=r.status_code,
        headers={
        "content-type":
        r.headers.get(
        "content-type",
        "application/json")
        }
    )
```

---

# 六、创建 compose

```
nano docker-compose.yml
```

内容：

```
services:

  ncm-adapter:

    build: .

    container_name: ncm-adapter

    restart: unless-stopped


    ports:

      - "2001:3000"


    volumes:

      - ./data:/data


    networks:

      - ncm-api_default



networks:

  ncm-api_default:

    external: true
```

---

# 七、启动 adapter

```
docker compose up -d --build
```

检查：

```
docker ps | grep adapter
```

---

# 八、测试 adapter

## 1. 登录接口

```
curl http://localhost:2001/login/qr/key
```

应该正常。

---

## 2. 搜索

```
curl \
"http://localhost:2001/search?keywords=%E5%91%A8%E6%9D%B0%E4%BC%A6"
```

---

## 3. 播放

第一次：

```
time curl \
"http://10.10.10.226:2001/song/url/v1?id=400164559&level=exhigh"
```

可能：

```
5-20秒
```

第二次：

```
time curl \
"http://localhost:2001/song/url/v1?id=400164559&level=exhigh"
```

应该：

```
0.05秒左右
```

日志：

```
docker logs ncm-adapter
```

看到：

```
CACHE HIT
```

---

# 九、Music Assistant 添加

MA：

```
音乐提供商
 ↓
添加
 ↓
NetEase Cloud Music
```

地址：

```
http://10.10.10.226:2001
```

不要：

```
/
```

不要：

```
/song
```

只填：

```
http://10.10.10.226:2001
```

然后：

二维码登录。

---

# 十、以后维护

## 查看日志

ncm-api:

```
docker logs -f ncm-api
```

adapter:

```
docker logs -f ncm-adapter
```

---

## 更新 ncm-api

```
cd ~/docker/ncm-api

docker compose pull

docker compose up -d
```

---

## 更新 adapter

```
cd ~/docker/ncm-adapter

docker compose up -d --build
```
