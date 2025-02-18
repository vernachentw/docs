---
title: GIPSense Map API 使用手冊(CHT)
tags: [Product, CHT]

---

# GIPSense Map API 使用手冊(CHT)
[TOC]

## 版本
| Updated Date | Version|Notes |
|--------------|-------|-------|
| 2023/01/29   | 1.0.0 |1st Release|
| 2023/01/29   | 1.1.0 |新增 api:<br>類別A：<br> isAccessReady<br> setPositions<br> setCurrentTarget<br>zoomToAll<br> zoomToSpace<br> zoomToAllPositions<br> setPositionDisplay<br> 類別B：<br> accessReady <br> currentTargetChanged|

## 前言
為滿足第三方應用整合定位地圖需求，並提供快速整合的開發工具，GIPSense 提供嵌入式圖台 API，可直接將地圖介面整合至應用系統之使用者介面，不必熟悉地圖介面開發框架。嵌入式圖台以 iframe 型式，提供開發者進行定位應用整合所需之地圖操作功能 API。並提供透過 channel 方式呼叫溝通或直接以 URL GET 參數進行控制。一般而言，GIPSense 所提供之嵌入式圖台之呼叫資源路徑為以下格式
```http=
http://{FQDN or IP}:8686/rtls/app/{lang}/map
```
參數說明：
- FQDN or IP：服務完整網域域名或 IP 位址
- lang：若為多語版本 GIPSense，則需於此處代入語系版本，如：zh-tw 或 en，如為非多語版本，則可省去此路徑

以下說明兩種介接方式。

## 方式一、使用 channel 方式呼叫與溝通
- message 格式： {type: string, cmd: string, data: object or string}
    - 參數：
        - 類型: 訊息類型, 包含以下: **request, response, info**
        - cmd: 指令名稱, 參考後續API指令內容
        - data: 資料，根據型態和指令名稱會有不同
- 類型：嵌入式圖台 API 提供兩類型如下：
    - **類型A(發出請求，取得回應)**： 由開發者控制，對圖台 app 發出一個 request message 後，app 回傳一個收到指令的 response message，可用於控制 app 行為，或取得 app 狀態
    - **類型B(取得圖台資訊)**： 由圖台 app 主動發出 message，通知開發者 app 已完成某項動作。
- 授權認證說明：
 於實際應用操作，由於圖資機敏性因素，於 GIPSense 後台可限定圖資是否限定授權瀏覽，若為限定授權之情況，則必須由 FAE人員提供核發之 API Key，並透過類型A中之「1.3 設定特殊資訊： configure」功能API 進行認證設定，始能呈現地圖之資料。

### 1. 類型A - 發出請求，取得回應
#### 1.1 確認 API 是否準備完成： isReady
- 格式： isReady()
- 說明： 確認 API 準備完成，才可以開始操作 API。
- Request 範例：
    ```javascript=
    window.postMessage({type: "request", cmd: "isReady", data: null})
    ```
- Response 訊息範例：
    ```json=
    {type: "response", cmd: "isReady", data: true}
    ```

#### 1.2 確認樓層的地圖資訊是否載入完成: isSpaceReady

- 格式： isSapceReady()
- 說明： 確認目前 space 是否已經地圖資訊載入完成。
- Request 範例：
    ```javascript=
    window.postMessage({type: "request", cmd: "isSpaceReady", data: null})
    ```
- Response 訊息範例：
    ```json=
    {type: "response", cmd: "isSapceReady", data: true}
    ```
#### 1.3 設定特殊參數: configure

- 格式：configure({key: string, value: unknown })
- 參數：
    - key： string - 欲執行操作類型key值
    - value： unknown - 欲執行操作類型所需設定的值，型態依不同設定內容而定
- 說明：
    - 設定特殊資訊，由此控制的設定會影響整個獨立圖台運作。
    - 設定開發者驗證用途：
        - key: authentication
        - value:{apiKey: string, appId: string}。
- Request 範例：
    - Setting up API token authentication information:
    ```json=
    window.postMessage({type: "request", cmd: "configure", data: {key: "authentication", value: {apiKey: "xxx", appId: "ooo"} }})
    ```
    :::info
    此操作於測試機環境不需執行認證作業，於正式環境發放API key
    :::

#### 1.4 設定欲顯示的標的物： setTargetItemUids
- 格式： setTargetItemUids(uids)
- 參數：
    - uids： Array(string) - 欲顯示標的物的 monitored item uid 陣列
    - 說明： 被指定的人員，點位才會顯示在地圖上。通常在收到 ready 訊息再呼叫。
- request 範例：
    - 所有人員： 
    ```javascript=
    window.postMessage({type: "request", cmd: "setTargetItemUids", data: null})
    ```
    - 指定兩個：
    ```javascript=
    window.postMessage({type: "request", cmd: "setTargetItemUids", data: ["P-00001", "P-00002"]})
    ```
- response 訊息範例
    ```json=
    {type: "response", cmd: "setTargetItemUids", data: ["P-00001", "P-00002"]}
    ```

#### 1.5 取得欲顯示的人員： getTargetItemUids
- 格式： getTargetItemUids()
- 說明： 回傳一個 uids 陣列，取得目前欲顯示的人員 itemUid。
- request 範例：
    ```javascript=
    window.postMessage({type: "request", cmd: "getTargetItemUids", data: null})
    ```
- response 訊息範例
    - 所有人員： 
    ```json=
    {type: "response", cmd: "getTargetItemUids", data: null}
    ```
    - 目前指定兩個：
    ```json=
    {type: "response", cmd: "getTargetItemUids", data: ["P-00001", "P-00002"]}
    ```
    
#### 1.6 設定欲顯示的標籤： setTargetTagIds

- 格式： setTargetTagIds(ids)
- 參數：
    - ids： Array(number) - 欲顯示標籤的 id 陣列
- 說明： 被指定的標籤，點位才會顯示在地圖上。通常在收到 ready 訊息再呼叫。
- request 範例：
    - 所有標籤： 
    ```javascript=
    window.postMessage({type: "request", cmd: "setTargetTagIds", data: null})
    ```
    - 指定兩個： 
    ```json=
    window.postMessage({type: "request", cmd: "setTargetTagIds", data: [44889, 46624]})
    ```
- response 訊息範例
    ```json=
    {type: "response", cmd: "setTargetTagIds", data: [44889, 46624]}
    ```

#### 1.7 取得欲顯示的標籤： getTargetTagIds

- 格式： getTargetTagIds()
- 說明： 回傳一個 ids 陣列，取得目前欲顯示的標籤 id。
- request 範例：
    ```javascript=
    window.postMessage({type: "request", cmd: "getTargetTagIds", data: null})
    ```
- response 訊息範例
    - 所有標籤： 
    ```json=
    {type: "response", cmd: "getTargetTagIds", data: null}
    ```
    - 目前指定兩個：
    ```json=
    {type: "response", cmd: "getTargetTagIds", data: [44889, 46624]}
    ```

#### 1.8 設定是否只顯示標的物點位： setItemOnly

- 格式：setItemOnly(value: boolean)
- 參數：
    - value: (boolean) - 決定是否只顯示標的物點位
- 說明： 設定是否只顯示標的物點位。
- request 範例：
    - 只顯示標的物：
    ```javascript=
    window.postMessage({type: "request", cmd: "setItemOnly", data: true})
    ```
    - 同時顯示標的物及標籤：
    ```javascript=
    window.postMessage({type: "request", cmd: "setItemOnly", data: false})
    ```
- response 訊息範例
    ```json=
    {type: "response", cmd: "setItemOnly", data: true}
    ```

#### 1.9 取得目前是否只顯示標的物點位： getItemOnly

- 格式：getItemOnly()
- 說明： 取得目前是否只顯示標的物點位。
- request 範例：
    ```javascript=
    window.postMessage({type: "request", cmd: "getItemOnly", data: null})
    ```
- response 訊息範例
    ```json=
    {type: "response", cmd: "getItemOnly", data: true}
    ```
#### 1.10 設定在無法定位時是否使用最後有效點位： setUseLastValidPosition

- 格式：setUseLastValidPosition(value: boolean)
- 參數：
    - value: (boolean) - 決定在無法定位時是否使用最後有效點位
- 說明：
    - 設定在無法定位時是否使用最後有效點位。
    - 離線的點位只能顯示最後有效點位，如果設為 false 則看不到離線點位。
- request 範例：
    - 無法定位時使用最後有效點位顯示：
    ```javascript=
    window.postMessage({type: "request", cmd: "setUseLastValidPosition", data: true})
    ```
    - 無法定位時不顯示點位：
    ```javascript=
    window.postMessage({type: "request", cmd: "setUseLastValidPosition", data: false})
    ```
- response 訊息範例
    ```json=
    {type: "response", cmd: "setUseLastValidPosition", data: true}
    ```

#### 1.11 取得目前在無法定位時是否使用最後有效點位： getUseLastValidPosition

- 格式：getUseLastValidPosition()
- 說明： 取得目前在無法定位時是否使用最後有效點位。
- request 範例：
    ```javascript=
    window.postMessage({type: "request", cmd: "getUseLastValidPosition", data: null})
    ```
- response 訊息範例
    ```json=
    {type: "response", cmd: "getUseLastValidPosition", data: true}
    ```
#### 1.12 設定地圖縮放尺寸： setZoom

- 格式：setZoom(zoom: number)
- 參數：
    - zoom: (number) - 縮放尺寸
- 說明： 指定地圖縮放尺寸，縮放至合適的顯示大小。
- request 範例：
    ```javascript=
    window.postMessage({type: "request", cmd: "setZoom", data: 22})
    ```
- response 訊息範例
  ```json=
  {type: "response", cmd: "setZoom", data: 22}
  ```

#### 1.13 取得地圖縮放尺寸： getZoom
- 格式：getZoom()
- 說明： 取得目前的地圖縮放尺寸。
- request 範例：
    ```javascript=
    window.postMessage({type: "request", cmd: "getZoom", data: null})
    ```    
- response 訊息範例
    ```javascript=
    {type: "response", cmd: "getZoom", data: 22}
    ```

#### 1.14 切換空間： changeSpaceAsync
- 格式： changeSpaceAsync(spaceName: string)
- 參數：
    - spaceName：(string) - 樓層名稱
- 說明： 根據指定的 space name 切換樓層，以非同步的方式載入該樓層的地圖資訊。
- request 範例：
    ```javascript=
    window.postMessage({type: "request", cmd: "changeSpaceAsync", data: "2F"})
    ```
- response 訊息範例：
    ```json=
    {type: "response", cmd: "changeSpaceAsync", data: "2F"}
    ```
    
#### 1.15 取得目前定位空間資訊： getCurrentSpace
- 格式：getCurrentSpace()
- 說明： 取得目前的樓層資訊，提供名稱以識別不同樓層。
- request 範例：
    ```javascript=
    window.postMessage({type: "request", cmd: "getCurrentSpace", data: null})
    ```
- response 訊息範例
    ```json=
    {type: "response", cmd: "getCurrentSpace", data: {name: "2F"}}
    ```
#### 1.16 定位指定目標： locateItem

- 格式： locateItem({itemUid: string, setCenter:boolean, setCurrent:boolean})
- 參數：
    - itemUid： (string) - 目標 uid
    - setCenter： (boolean) - 是否依該目標為中心移動地圖
    - setCurrent： (boolean) - 是否設為目前點位 (顯示 popup)
- 說明： 用來定位某一個 MonitoredItem 的點位，根據點位所在樓層切換 space 並顯示點位。
- request 範例：
    ```javascript=
    window.postMessage({type: "request", cmd: "locateItem", data: {itemUid: "P-00001", setCenter: true, setCurrent: true}})
    ```
- response 訊息範例：
    ```json=
    {type: "response", cmd: "locateItem", data: {itemUid: "P-00001", setCenter: true, setCurrent: true}}
    ```

### 2. 類型B - 取得資訊

#### 2.1 API 準備完成通知： ready

- 說明： 當 API 準備完成，會自動發出通知，以確保 app 不會過早操作 API。
- info 訊息範例：
    ```json=
    {type: "info", cmd: "ready", data: true}
    ```

#### 2.2 樓層切換通知：spaceChanged

- 說明： 當樓層切換時會自動發出通知，以確保 app 能同步樓層資訊。
- info 訊息範例：
    ```json=
    {type: "info", cmd: "spaceChanged", data: {name: "2F"}}
    ```


#### 2.3 樓層準備完成通知： spaceReady

- 說明： 當樓層切換後，載入地圖圖層資訊完成，會自動發出通知，以確保 app 能同步地圖圖層資訊。請注意如果快速切換樓層，只會載入最後一次切換樓層的地圖圖層資訊。
- info 訊息範例：
    ```json=
    {type: "info", cmd: "spaceReady", data: {name: "2F"}}
    ```
    
#### 2.4 標的物點選通知： itemClicked

- 說明： 在地圖上點選標的物後，標的物上方會出現白底提示框及藍色標的物名稱，再點選藍色的標的物名稱就會發出通知，通知的內容為標的物的屬性內容。
- info 訊息範例：
    ```json=
    {type: "info", cmd: "itemClicked", data: {"uid":"P-00001","kind":"MEMBER","name":"Guest1", ... }}
    ```

## 方式二 使用HTTP GET方式呼叫
使用HTTP GET介面方式呼叫嵌入式地圖所提供之參數功能如下整理：

- Request URI
```htmlembedded=
http:// {IP or FQDN}:8686/rtls/app/{lan}/map
```
- Method: GET
- Request Header: accept application/json
- 參數：
    - mode: 地圖模式, 包含以下參數:
        - default: 預設模式, 需透過變更space name來切換地圖
        - locating: 追蹤模式, 追蹤指定的itemUid，圖台自動依據item的位置來切換地圖
    - space: 空間名稱. 此參數當**mode = default**時有效，接收參數為GIPSense中所設定之space 名稱
    - locatingItemUid: 追蹤標的物識別碼. 此參數當**mode = locating**時有效，接受參數為定為系統中，標的物之Uid

:::info
執行此方式呼叫，仍須對地圖視窗物件執行認證設定：
```json=
window.postMessage({type: 'request', cmd: 'configure', data: {key: "authentication", value: {apiKey: "xxx", appId: "xxx"}}}, '*');
```
:::