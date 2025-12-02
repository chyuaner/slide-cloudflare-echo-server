Yuan 的 HTTP Echo Server (Slide)
===

* 本投影片網址: <https://yuaner.tw/slide-cloudflare-echo-server/>
* 專案網址: <https://github.com/chyuaner/cloudflare-echo-server>

這個Repo是我做這個專案的心得整理

## 以下是這個專案的簡介

本專案是提供CDN Edge層級的http回音鸚鵡伺服器（也提供本機端獨立運作的方式）

就如字面上所說的，你對我發出的Request後，我的伺服器就會把我從你這邊接到的資訊：以什麼網址字串、帶了什麼Post Body、Request Header，一五一實的以ResponseBody方式回應給你。

同時也可以當作MyIP查詢使用，會顯示在「Host」區塊。


## 投影片採用模板
* 官網： <https://marp.app/>
* 相關語法： <https://marpit.marp.app/>
* 自動佈署模板： <https://github.com/robalexdev/marp-to-pages>
