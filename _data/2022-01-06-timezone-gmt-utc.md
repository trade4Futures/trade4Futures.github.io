---
title: "時區(GMT, UTC, CST)？時間戳(timestamp)？時間格式(ISO)？時間處理一次看懂"
date: "2022-01-06"

tags:
    - JavaScript
    - timezone

excerpt: "
    時間處理是實務上很常遇到的情況，不論是紀錄log，或是寫一個搶票系統。但剛觸碰時怎麼時區那麼麻煩？
    有 GMT, UTC 又有 CST，三者差在哪？怎麼 ISO 時間格式好奇怪？怎麼前後端時間不一致ＱＱ
    本篇就來了解時間處理的基本概念吧！
    "
---

## 本篇重點

- 介紹 GMT, UTC, CST 時區表示的差異。
- 介紹時間戳(timestamp, epoch)。
- 介紹**標準的時間表示**方法(ISO 表示法)。
- 介紹 JS 原生 `Date` 一些輸出解釋。
- 看一下 JS 時間處理套件 `spacetime` 的厲害。

## 問題背景

- 最近遇到時間同步的功能開發，前端可以透過手動設定時間，也可以要求後端採用 NTP 網路校時。
- 但可能遇到前端時區是莫斯科，但後端是台北。也就是後端時間會比較快。
- 如果產品又有使用者設定排程功能，可能前端設定 10:00 執行，結果前端 07:00 時，排程就開始跑了(因為後端已經到 10:00)。
- 為了避免上述時間不一致的狀況，必須確保前後端時間、時區一致。

## 時間戳(timestamp, epoch)

- 提到時間一致，很難不談時間戳(timestamp 或稱 epoch)。
- 時間戳單位是「毫秒」或「秒」，是從格林威治 `1970-01-01 00:00:00` 開始計算的。
    
    ```jsx
    // 例如 2022-01-01 00:00:00 與 2022-01-01 00:00:00 的 timestamp 相差 1000
    timeA = new Date('2022-01-01 00:00:00')
    timeB = new Date('2022-01-01 00:00:01')
    
    timeB.getTime() - timeA.getTime() // 1000ms
    ```
    
- **時間戳沒有時區概念！**
    - 不論是莫斯科還是台北，在同一時間下，會拿到相同的時間戳。
    - 這邊介紹好用的 js 時間處理套件: [spacetime](https://www.npmjs.com/package/spacetime)。用這個套件來做介紹：
    
    ```jsx
    // 看看當前時刻，在不同時區的時間戳
    LondonNow = spacetime.now().goto('Europe/London')
    MoscowNow = spacetime.now().goto('Europe/Moscow')
    TaipeiNow = spacetime.now().goto('Asia/Taipei')
    
    // 檢視時間戳會發現同一個時刻都相同
    LondonNow.epoch
    MoscowNow.epoch
    TaipeiNow.epoch
    
    // 但相同的時間戳會依照當地時區轉換成對應的當地時間
    const timestamp = TaipeiNow.epoch
    const YYYYMMDDHHmmss = '{iso-short} {hour-24-pad}:{minute-pad}:{second-pad}'
    const Taipei = spacetime(timestamp).goto('Asia/Taipei').format(YYYYMMDDHHmmss)
    const London = spacetime(timestamp).goto('Europe/London').format(YYYYMMDDHHmmss)
    const Moscow = spacetime(timestamp).goto('Europe/Moscow').format(YYYYMMDDHHmmss)
    ```
    

## 時區(GMT, UTC, CST？)

- 常見的時區表示方式有 GMT, UTC, CST，那麼三者有什麼差異呢？
- GMT：格林威治時間，小學教的就是這個，而格林威治 GMT+0000，台北 GMT+0800。
- **UTC**：世界協調時間，**與 GMT 差異不大，但為目前國際較常用的格式，**格林威治一樣 UTC+0000，台北 UTC+0800。
- CST：國家標準時間，**基本上不採用這種格式**，因為不知道「國家」是指哪個國家。例如可以採用美國、北京、澳洲。

### GMT, UTC 與 CST 換算關係？

- GMT 基本上可以跟 UTC 互換。例如台北時區可以表示成 GMT+0800、UTC+0800。
- CST 則要看採用的國家，例如台灣常用的是「中原標準時間」，也就是北京時間，也就是 `CST(北京) = GMT+0800 = UTC+0800`。

### ISO 時間表示格式

- 參考[官方說明](https://www.iso.org/iso-8601-date-and-time-format.html)或是參考 [RFC 文件說明](https://datatracker.ietf.org/doc/html/rfc3339#section-5.6)。
- ISO 格式：`YYYY-MM-DDTHH:mm:ss.msZ` 或 `YYYY-MM-DDTHH:mm:ss.ms時區`
    - `T` 是劃分「日期」與「時間」的記號。
    - `Z` 表示 `UTC+0` 的意思。**若看到 ISO 表示法中帶有** `Z`**，表示這是** `UTC+0` **的時間**。
    - `時區` 就是採用的時區，例如台灣就是 `YYYY-MM-DDTHH:mm:ss.ms+08:00`。
    
    ```jsx
    // 以下採用 spacetime 套件
    // 例如當前時間
    now = spacetime('2021-01-06 14:47:40').goto('Asia/Taipei')
    
    // 將「現在時間」顯示成 ISO 表示法，以下分別表示 UTC+0 的 ISO 時間，與台北的 ISO 時間
    // 注意到兩者時間會差 8 小時
    now.format('iso_utc')  // 2021-01-06T06:47:40.000Z
    now.format('iso')      // 2021-01-06T14:47:40.000+08:00
    ```
    

### JS 中的顯示方式

- 原生 `Date()` 用法：
    
    ```jsx
    // 顯示當前時間
    now = new Date('2021-01-06 14:47:40')
    
    // 依照地區輸出時間結果(預設顯示系統語系、12小時制)
    now.toLocaleString()         // '2022/1/6 下午2:47:40'
    now.toLocaleString('en-US')  // '1/6/2022, 2:47:40 PM'
    now.toLocaleString('en-US', {hour12: false})  // '1/6/2022, 14:47:40'
    
    // 查看該時間轉成 UTC+0 的時間
    now.toUTCString()  // 'Wed, 06 Jan 2021 06:47:40 GMT'
    
    // 查看該時間的 ISO 表示法
    now.toISOString()  // '2021-01-06T06:47:40.000Z'
    ```
    

## 前端提供時間、時區，要求後端設定成對應的時間。

- 前端給 `timestamp`、`timezone`，後端接收後。
- `timezone` 給**地區字串**(例如`'Asia/Taipei'`)，方便利用 `spacetime.goto()` 轉換出對應時區的時間。
    
    ```jsx
    // 例如前端是俄羅斯(GMT+0300)，後端是台北(GMT+0800)
    // 假設俄羅斯時間是 2021-01-05 09:31:17
    // 則台北時間是    2021-01-05 14:31:17

    // 前端執行以下內容，獲得莫斯科當前時刻及其時間戳
    Moscow = new Date()
    MoscowTimestamp = Moscow.getTime() // 1641364307150
    MoscowTimezone = 'Europe/Moscow'
    ```
    
    ```jsx
    // 後端透過 timestamp 與 時區，推算出莫斯科的當地時間
    MoscowLocalTime = spacetime(MoscowTimestamp).goto(MoscowTimezone)
    ```
    
- `timezone` 給 GMT (例如 GMT+0500)，直接把 `timestamp` 與後端的時區做加減。
    
    ```jsx
    // 例如前端是俄羅斯(GMT+0300)，後端是台北(GMT+0800)
    // 假設俄羅斯時間是 2021-01-05 09:31:17
    // 則台北時間是    2021-01-05 14:31:17

    // 前端執行以下內容，獲得莫斯科當前時刻及其 timestamp
    Moscow = new Date()
    MoscowTimestamp = Moscow.getTime() // 1641364307150
    ```
    
    ```jsx
    // 前端要改寫後端時間，則將其 timestamp、GMT+0300 送到後端
    // 因為後端快了 5 小時，所以後端拿到 timestamp 後，減去 5 * 60 * 60 * 1000 可推出莫斯科的時間

    // 後端從 timestamp 及時區差值推算出莫斯科的時間，但後端仍採用台北時區
    MoscowLocalTime = new Date(MoscowTimestamp - 5 * 60 * 60 * 1000) // 2021-01-05 09:31:17
    
    // 以下是莫斯科提供的時間戳，對應到的台北時間，可以看到時間快了 5 小時
    TaipeiActual = new Date(MoscowTimestamp) // 2021-01-05 14:31:17
    ```
    

> 💡 如果後端有使用 `timedatectl` 時間管理工具，則 `timezone` 用地區字串比較方便。
> 如果後端使用 `ntp` 網路校時，還是要提供時區。

## 結語
* 時間戳(timestamp, epoch)沒有時區概念，單位是「秒」或「毫秒」。
* 同一時刻下，世界各地拿到的時間戳相同，但相同的時間戳可以藉由時區差別轉換成當地時間。
* GMT 基本上與 UTC 相同，CST 則是 UTC+8。
* ISO 是標準的時間表示法，若出現 `Z` 表示該時間為 UTC+0 的時間。
* 最後，關於前後端時間一致的處理並沒有說得太詳細，之後有整理出一套比較完善的流程再寫上來囉！

## 參考網站
