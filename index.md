---
marp: true
title: Yuan 的 HTTP Echo Server
image: img/cover.png
theme: default
paginate: true
footer: 
style: |
    section.lead h1, section.lead h2 {
        text-align: center;
    }
    /*img {
        height: 14rem;
    }*/
    .columns {
        display: grid;
        grid-template-columns: repeat(2, minmax(0, 1fr));
        gap: 1rem;
    }

---
<!-- _paginate: skip -->
<!-- ![bg right 65%](img/qrcode_url.svg) -->

# Yuan 的 HTTP Echo Server


*Yuan Chiu <chyuaner@gmail.com>*

-------------------------------------------------

# 主要核心功能

![bg right:50% 100%](img/screenshot.png)

**本專案是提供CDN Edge層級的http回音鸚鵡伺服器**

你對我發出Request，我的伺服器就會把我從你這邊接到的資訊：

* 以什麼網址字串
* 帶了什麼Post Body
* 帶了什麼Request Header

一五一實的以ResponseBody方式回應給你。

同時也可以當作MyIP查詢使用，會顯示在「Host」區塊。

---
<!-- _class: lead -->
# ➡️ 專案特色

---

# 在CDN Edge層級直接提供完整服務

### 傳統後端架構圖（以我習慣的PHP為例）
```
公網 → CDN快取 → Reverse Proxy(管理vhost) → Nginx → php-fpm → 我的專案
```

### CDN層級**直接服務**架構圖
```
公網 → CDN直接服務
```

理論上極致效能低延遲

---
# JSON格式作為預設Response Body輸出

![bg left 105%](img/postman.png)

基本上以 ealen/echo-server 專案為基礎，重新撰寫復刻核心功能

分為三大塊：
* request
* http
* host

可用於Postman、Paw、Insomnia、Hoppscotch等HTTP API調試客戶端使用

---

# ➡️ 有設計精美的網頁UI界面
![bg right:63% 100%](img/screenshot.png)

---

# 清楚明瞭的HTTP Method

網址列顏色會根據 GET, POST, PUT, DELETE改變顏色

![GET](img/GET.png)
![POST](img/POST.png)
![PUT](img/PUT.png)
![DELETE](img/DELETE.png)

顏色挑選有特別與Swagger、API Doc對齊

---

# 網址文字友善複製

![](img/query-copy.png)

有特別為 **「URL Params」、「URL Query」區塊特別設計友善文字複製** 。界面乍看下是ul li項目清單，但圈選文字後，會直接複製成可直接貼上網址列的字串

---
# RWD手機友善 與 Dark Mode配色
<!-- _backgroundColor: black -->
<!-- _color: white -->
<div class="columns">
<div id="left">

## RWD手機友善
![h:12cm](img/mobile.png)


</div>

<div id="right">

## Dark Mode配色
![](img/dark-mode.png)

</div>

---
# 網頁界面仍有提供Raw Response Body
模擬本後端的JSON模式原始輸出，方便切換客戶端比對使用


![h:12cm](img/raw.png)


---
# 效能考量設計
![h:9cm](img/one-request.png)

**網頁版不會產生額外Request載入其他資源！（如:圖片、CSS、JS等等）**

* 所有外部資源如Icons與Syntax Highlight JS都已直接內嵌在單一這個Request。
* 所有圖片都是SVG格式


---
# ➡️ 附加功能

![bg right 95%](img/Screenshot_20251202_074531%20(Edit).png)

提供 URL Query 參數 來控制以下功能
* `echo_code=200`： 控制要回傳的 HTTP Status Code
* `echo_time=3000`： 控制伺服器要延遲多久才會送 Response ，模擬較差網路品質狀況使用

---

# ➡️ 提供其他本地架設方式
![bg right 95%](img/docker-hub.png)
本專案雖然是為Cloudflare量身打造的，但也有提供傳統的架設方式

* Docker
    `docker run -p 3000:3000 chyuaner/echo-server`
* Node直接啟動
    `npm i && npm run start`

-------------------------------------------------
<!-- _class: lead -->
# ➡️ 開發心得

---

# 本次採用語言

其實我本身比較擅長PHP，但...
* 本次目的就是想減少自有主機依賴，變成要遷就雲端提供商的限制
* Cloudflare Workers基本上就是以NodeJS語言為主
* 本次目的是盡可能減少線路「繞圈」與「轉換」，來換取效能

---

# 開發緣由
## 先前都是使用 ealen/echo-server

雖然已包好Docker，架設簡單、輸出結構簡潔漂亮，但是...
* 只有提供JSON格式回傳
* 需要有自己的主機架設
    * 然後我近期的主機資源很吃緊~~~
* 其他原因，下述...

---

# 近期對CDN優化有興趣
我以前使用Cloudflare Worker用途
* 網址搬遷時用白名單Mapping表概念，讓特定舊網址跳轉到新網址
* 短網址直接跳轉
* 偵測客戶端特定國家自動跳轉專用網址

---

# 後來發現Workers其實可以做到以下事情

* 拿到客戶端傳來的Request
    * **URL Params / Query**
    * **Request Post Body**
    * **Request Post Body**
* 可以控制Response內容
    * 跳轉導向到特定的網址
    * 改寫 Response Body內容
    * **可以直接輸出指定的字串，也包含整串HTML大字串**

🚀 **評估下來，本次需求僅用以上功能就足夠了！！！**

---

# Cloudflare Worker執行能力有限
* 並不是完整的NodeJS環境，只能執行Cloudflare API名單內的function
* 沒有檔案概念，若需要額外資源檔案，需要另外加掛資源空間
    * 也直接導致很難使用ejs之類的模板框架（遇到底層有使用到`eval`與`fs`就會被拒絕執行）

## 對策：
* 所有資源全部Hard Code寫死內嵌，整理成JS陣列。
* 不使用模板系統，直接靠func整理HTML字串。
    * 但自己在規劃JS Function時仍然會盡量模組化。

---

# 提供其他本地架設方式
開發過程90%都是使用Cloudflare提供的wrangler模擬環境測試
但考量隨時會有獨立架設需求，仍有特別製作 `/src/server.js` 供獨立執行使用

### `/src/index.js` 核心主功能流程與Cloudflare Workers直接進入點
### `/src/server.js` 獨立執行專用 進入點
* 以NodeJS最原生的node:http為主
    * 本次需求沒有複雜的 router、session...等等 ，也沒有多頁面需求，不使用express這類框架
* 會銜接回 `/src/index.js` ，讓所有功能都與Cloudflare Workers模式對齊
    * Cloudflare專屬功能提供的GeoIP資料則無法使用，需要用替代方式處理GeoIP問題。因為我暫時還沒有常駐用的獨立架設需求，就暫時不處理了。


---

## 順便練習Docker建置

---

# 搭配 Github Actions 自動更新Docker Hub

---

## 後續新專案要不要繼續用Cloudflare Worker？
* Cloudflare也有提供
* 大前提：Cloudflare Workers執行能力有限！！！
    * 本次專案已經用了大量的Wordaround方法（尤其是有不少直接Hard Code寫死）

* 如果是傳統電商需求，100%一定不會考慮！！
* fake img 復刻中

---

# 就醬，歡迎交流😀