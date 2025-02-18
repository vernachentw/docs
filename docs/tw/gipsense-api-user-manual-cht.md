---
title: GIPSense API 使用手冊(CHT)
tags: [Product, CHT]

---

# GIPSense API 使用手冊(CHT)
[TOC]

## 升版歷程
| Updated Date | Notes |
|--------------|-------|
| 2023/01/19   | 初版   |
| 2024/08/09   | 加入B項定位串接系統常見情境補充   |
| 2025/02/10   | 補充綁定流程情境   |



## A. 資源說明
:::success
1. 此份文件說明GIPSense主要API使用之入門介紹，完整API之使用請用戶參考Swagger對應文件內之說明
2. 後續文件內容之demoIP可替換為使用者自行架設之環境或由GIPS所提供之測試環境
3. demoIP Sandbox測試：uwbdemo1.gips.com.tw
:::


### 1.GIPSense 管理平台
 - http://{demoIP}:8686/rtls/app 
 - 帳密:admin/!p@ssw0rd
 - GIPSense 管理平台提供完善定位場景所需之基礎應用情境功能,定子圍籬、黑/白名單、歷史軌跡、事件告警、標籤下發控制、攝影機連動等功能

### 2. GIPSense Swagger API
#### 2.1 認證API Swagger
 - http://{demoIP}:8686/rtls/rest/auth/1.0.0/docs.html
 - 提供認證相關 API 規格
 - 需先進行認證後才可瀏覽2.2之功能API Swagger頁或進行功能API之操作

#### 2.2 GIPSense 功能API Swagger
 - http://{demoIP}:8686/rtls/rest/1.0.0/docs.html
 - 提供所有GIPSense平台功能API 規格
 - 須為登入/認證的狀態下才可瀏覽

### 3. GIPSense 嵌入式 Map API
 - http://{demoIP}:8686/rtls/app/zh-tw/map
 - 嵌入式圖台為一網頁形式頁面，透過開發客製化事件功能，提供開發人員可利用操作DOM中之window物件與圖台進行互動

## B. 串接定位系統情境說明
本系統API使用需先經過Auth API取得cookie認證或於Header帶入下方API 金鑰參數始可操作其他功能API。

:::success
x-api-key: {由相關人員提供}
x-app-id: {由相關人員提供}
:::

### 1. 資料串接 & 定位標籤應用
在GIPSense系統中，追蹤目標與定位標籤為兩種不同的單元，一般而言追蹤目標的資料會來自於應用客戶端的資料，如：人員/員工清單、資產設備清單、車輛/推高機等設備清單。定位系統中必須預先建立所要追蹤目標的清單(可透過介面匯入或API建立)。
而UWB定位標籤需與追蹤目標進行綁定，系統才會將定位標籤之定位結果賦予給追蹤目標。

#### 1-1. 標的物資料建立：將客戶資料建立至定位系統
本情境主要參考章節C中 功能API - MonitoredItem 標的物群組，主要使用到API列舉以下
  - C-3.3 建立標的物: POST /rest/1.0.0/items
  - C-3.5 刪除標的物: DELETE /rest/1.0.0/items/{uid}

:::warning
1. 需確認所使用之itemUid為何，此作為雙方系統後續串接之主Key
2. 定位系統可設定不同的標的物類型，請評估是否有需要建立，如客戶、服務人員
3. displyName欄位為屆時地圖上顯示之名稱，須考量代入之需求
:::

#### 1-2. 標的物與標籤綁定/解綁定
本節情境以醫美客戶將標籤配發給來訪客戶為例，其流程為掃描定位標籤上之RFID再與來訪人員的識別碼進行綁定動作。如用戶為直接掃描定位標籤上之標籤ID，則可略過以RFID查詢定位標籤ID

  - C-2.1 批次查詢標籤: POST /rest/1.0.0/tagInfo/query (將RFID Reader所讀到之RFID碼，查詢得到UWB標籤ID)
  - C-3.6 綁定標的物: PUT /rest/1.0.0/items/{uid}/boundTagInfo
  - C-3.8 解除綁定標的物: PUT /rest/1.0.0/items/{uid}/boundTagInfo/clear
  - 綁定流程圖:
    ![](https://hackmd.io/_uploads/HyXWm5IZ6.png)
  - 解綁定流程圖:
    ![](https://hackmd.io/_uploads/ryvb7qLWp.png)

### 2. 取得標的物/標籤 即時位置
透過此API可直接取得標的物/標籤之即時狀態，如所在區域、電池電量、坐標、墊子圍籬狀態、手錶生理量測資訊等。

- C-4.1 批次查詢標的物狀態: POST /rest/1.0.0/targetState/batchQuery 
  ```json=
   //參考查詢範例，撈全部的標的物與標籤
   [
      {
        "targets": {
          
        },
        "conds": {
        },
        "countOnly": false
      }
    ]
  ```
## C. API 使用說明

### 1. API 開發驗證
#### 1.1 透過 Auth API 取得 Cookie
- POST /rest/auth/1.0.0/signin 
    - u: 帳號
    - p: 密碼
- 程式開發上，可透過 HttpCookieManager 管理 response中的 Cookie 值，並將 Cookie中的 SESSION 值帶入後續功能API 操作

#### 1.2 預設提供之API Key
在每個 API Request 之 header 帶入下列 key-value
- x-api-key: {另外提供}
- x-app-id: {另外提供}

### 2. 功能API - TagInfo 標籤群組
對應 GIPSense管理平台 **標籤列表** 功能頁，提供定位標籤建立(上架)、編輯、刪除等API操作功能

#### 2.1 批次查詢標籤:POST /rest/1.0.0/tagInfo/query
- Content-type: application/json
- Request Body Example:
```json=
{
  "query": {
    "bound": false,
    "tagIds": [
      42090
    ],
    "rfidIdCond": {
      "text": "xxxxxxxx",
      "partial": false
    },
  },
  "paging": {
    "page": 0,
    "pageSize": 10
  },
  "sort": [
    {
      "prop": "tagId",
      "dir": "ASC"
    }
  ]
}
````
- Request Parameters
    - paging:分頁資訊，不指定時表示不分頁
    - paging.page:分頁頁碼，第一頁頁碼為0，預設為0
    - paging.pageSize:分頁大小，0或負值以及不指定時表示不分頁
    - query:查詢條件，不指定時表示全部搜尋
    - query.bound:標籤是否為綁定狀態(true/false)，不指定表示不限制
    - query.tagIds: 指定查詢標籤識別碼，為十進制，可設定查詢多組
    - query.rfidIdCond.text: rfid數值字串
    - query.rfidIdCond.partial: rfid是否為完全匹配
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": {
    "total": 1,
    "page": 0,
    "pageSize": 10,
    "totalPages": 1,
    "content": [
      {
        "tagId": 44889,
        "height": 120,
        "displayName": "測試標籤",
        "icon": "{\"type\":\"fa\",\"data\":{\"prefix\":\"fas\",\"name\":\"genderless\",\"size\":16,\"ver\":\"2\"}}",
        "modelId": "GT100",
        "nfcId": null,
        "rfidId": null,
        "positioningConfig": null,
        "hardwareConfig": null,
        "cpxDeviceType": 0,
        "cpxDeviceId": "af59"
      }
    ],
    "count": 1
  }
}
```
#### 2.2 以TagID查詢標籤:POST /rest/1.0.0/tagInfo/queryByTagIds
- Content-type: application/json
- Request Body Example:
```json=
[44889]
````
- Request Parameters
tagIds:為以字串陣列輸入欲查詢之標籤十進制識別碼
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": [
    {
      "tagId": 44889,
      "height": 120,
      "displayName": "測試標籤",
      "icon": "{\"type\":\"fa\",\"data\":{\"prefix\":\"fas\",\"name\":\"genderless\",\"size\":16,\"ver\":\"2\"}}",
      "modelId": "GT100",
      "nfcId": null,
      "rfidId": null,
      "positioningConfig": null,
      "hardwareConfig": null,
      "cpxDeviceType": 0,
      "cpxDeviceId": "af59"
    }
  ]
}
```

#### 2.3 建立標籤:POST /rest/1.0.0/tagInfos
- Content-type: application/json
- Request Body Example:
```json=
{
  "tagId": 1111,
  "height": 120,
  "displayName": "測試標籤",
  "icon": null,
  "modelId": "GT100",
  "nfcId": "11111111",
  "rfidId": "111111111",
  "positioningConfig": null,
  "hardwareConfig": null
}
````
- Request Parameters
    - tagId: 標籤識別碼(十進制)
    - height: 標籤預設配置高度，建議值為120公分
    - displayName: 標籤顯示名稱
    - icon: 標籤於地圖上顯示樣式
    - modelId: 標籤配置型號，目前預設為GT100
    - nfcId: 配置於標籤上之NFC晶片識別碼，若無輸入則為null
    - rfidId: 配置於標籤上之RFID晶片識別碼，若無輸入則為null
    - positioningConfig: 進階設定，標籤定位參數設定 (可先帶null)
    - hardwareConfig: 進階設定，標籤硬體參數設定 (可先帶null)
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": {
    "tagId": 1111,
    "height": 120,
    "displayName": "測試標籤",
    "icon": null,
    "modelId": "GT100",
    "nfcId": "11111111",
    "rfidId": "111111111",
    "positioningConfig": null,
    "hardwareConfig": null,
    "cpxDeviceType": 0,
    "cpxDeviceId": "0457"
  }
}
```

### 3. 功能API - MonitoredItem 標的物群組
對應 GIPSense管理平台 **監控內容設定** 功能頁，提供監控內容(如醫療人員、設備)等資料之建立、刪除等管理，並且綁定/解綁定定位標籤之操作

#### 3.1 批次查詢標的物: POST /rest/1.0.0/item/query
- Content-type: application/json
- Request Body Example:
```json=
///完整格式
{
  "paging": {
    "page": 0,
    "pageSize": 10
  },
  "query": {
    "boundTagIds": [
      0
    ],
    "bound": true,
    "kinds": [
      "string"
    ],
    "keywordCond": {
      "text": "string",
      "partial": true
    },
    "cchStatus": "string"
  },
  "sort": [
    {
      "prop": "property1",
      "dir": "ASC",
      "ignoreCase": false,
      "nullHandling": "NATIVE"
    }
  ]
}
///範例
{
  "paging": {
    "page": 0,
    "pageSize": 10
  },
  "query": null
  },
  "sort": [
    {
      "prop": "uid",
      "dir": "ASC",
      "ignoreCase": false,
      "nullHandling": "NATIVE"
    }
  ]
}
````
- Request Parameters
    - paging:分頁資訊，不指定時表示不分頁
    - paging.page:分頁頁碼，第一頁頁碼為0，預設為0
    - paging.pageSize:分頁大小，0或負值以及不指定時表示不分頁
    - query:查詢條件，不指定時表示全部搜尋
    - query.boundTagIds:以標的物綁定之標籤識別碼作為查詢條件，不指定表示不限制
    - query.bound:標的物是否為綁定狀態(true/false)，不指定表示不限制
    - query.kinds:標的物類型(人員/設備)，不指定表示不限制
    - query.cchStatus:指定查詢彰基設備狀態字串(OR/外站/外借/維修)
    - keywordCond:關鍵字搜尋條件
    - keywordCond.text:關鍵字搜尋文字
    - keywordCond.partial:關鍵字搜尋是否為部分符合(true/false)
    - sort:搜尋結果排序設定,null即表示無設定回傳排序
    - sort.prop:排序依據欄位
    - sort.dir:排序方向(ASC/DESC)
    - sort.ignoreCase:排序是否區分大小寫
    - sort.nullHandling:當排序欄位為空值之處理機制(NATIVE, NULLS_FIRST, NULLS_LAST)
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": {
    "total": 1,
    "page": 0,
    "pageSize": 10,
    "totalPages": 1,
    "content": [
      {
        "sn": 1,
        "uid": "A000001",
        "kind": "EQUIPMENT",
        "name": "手術床A000001",
        "displayName": "手術床",
        "icon": "{\"type\":\"fa\",\"data\":{\"prefix\":\"fas\",\"name\":\"bed\",\"size\":16,\"ver\":\"2\"}}",
        "phone": "",
        "email": "",
        "category": "",
        "org": "",
        "creationtime": "",
        "boundTagId": 44889,
        "boundTime": "2020-07-21T05:44:02.976Z",
        "boundHeight": null
      }
    ],
    "count": 1
  }
}
```
#### 3.2 以Uid查詢標的物: POST /rest/1.0.0/item/queryByUids
- Content-type: application/json
- Request Body Example:
```json=
[
  "A000001"
]
````
- Request Parameters
uids:為以字串陣列輸入欲查詢之標的物識別碼
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": [
    {
      "sn": 1,
      "uid": "A000001",
      "kind": "EQUIPMENT",
      "name": "手術床A000001",
      "displayName": "手術床",
      "icon": "{\"type\":\"fa\",\"data\":{\"prefix\":\"fas\",\"name\":\"bed\",\"size\":16,\"ver\":\"2\"}}",
      "phone": "",
      "email": "",
      "boundTagId": 44889,
      "boundTime": "2020-08-17T08:40:04.288Z",
      "boundHeight": 0
    }
  ]
}
```
#### 3.3 建立標的物: POST /rest/1.0.0/items
- Content-type: application/json
- Request Body Example:
```json=
///依據彰基UML擴充
{
  "uid": "string",
  "kind": "string",
  "name": "string",
  "displayName": "string",
  "icon": "string",
  "phone": "string",
  "email": "string",
  "category": "string",
  "org": "string",
  "creationtime": DateTime,
}
````
- Request Parameters
    - uid:標的物識別碼
    - kind:標的物種類
    - name:標的物名稱
    - displayName:標的物顯示名稱
    - icon:標的物於地圖上之顯示icon
    - phone:標的物聯絡電話屬性
    - email:標的物聯絡郵件屬性
    - category:標的物子類別，依據kind有不同CodeList
    - org: 人員所屬廠商名稱
    - brand:設備所屬廠牌名稱
    - model:設備所屬型號
    - division:設備所屬科別
    - admin:設備所屬管理者
    - cchStatus:設備院內狀態(OR/眼科/維修/外借)
    - creationTime:標的物建立時間
    
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": {
    "total": 1,
    "page": 0,
    "pageSize": 10,
    "totalPages": 1,
    "content": [
      {
        "sn": 1,
        "uid": "A000001",
        "kind": "EQUIPMENT",
        "name": "手術床A000001",
        "displayName": "手術床",
        "icon": "{\"type\":\"fa\",\"data\":{\"prefix\":\"fas\",\"name\":\"bed\",\"size\":16,\"ver\":\"2\"}}",
        "phone": "",
        "email": "",
        "boundTagId": 44889,
        "boundTime": "2020-07-21T05:44:02.976Z",
        "boundHeight": null
      }
    ],
    "count": 1
  }
}
```
#### 3.4 更新標的物屬性: PUT /rest/1.0.0/items/{uid}
- Content-type: application/json
- Path Parameters:
    - uid: 欲更新標的物之識別碼
- Request Body Example:
```json=
///目前規格為標準產品之結構，彰基後續將依據訂定標的物UML擴充
{
  "displayName": "string",
  "email": "string",
  "icon": "string",
  "kind": "string",
  "name": "string",
  "phone": "string"
}
````
- Request Parameters
    - kind:標的物種類
    - name:標的物名稱
    - displayName:標的物顯示名稱
    - icon:標的物於地圖上之顯示icon
    - phone:標的物聯絡電話屬性
    - email:標的物聯絡郵件屬性
    
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": {
    "total": 1,
    "page": 0,
    "pageSize": 10,
    "totalPages": 1,
    "content": [
      {
        "sn": 1,
        "uid": "A000001",
        "kind": "EQUIPMENT",
        "name": "手術床A000001",
        "displayName": "手術床",
        "icon": "{\"type\":\"fa\",\"data\":{\"prefix\":\"fas\",\"name\":\"bed\",\"size\":16,\"ver\":\"2\"}}",
        "phone": "",
        "email": "",
        "boundTagId": 44889,
        "boundTime": "2020-07-21T05:44:02.976Z",
        "boundHeight": null
      }
    ],
    "count": 1
  }
}
```

#### 3.5 刪除標的物: DELETE /rest/1.0.0/items/{uid}
- Path Parameters:
    - uid: 欲刪除標的物之識別碼
- Response
```json=
{
  "success": true,
  "msg": "string",
  "data": {}
}
```
#### 3.6 綁定標的物: PUT /rest/1.0.0/items/{uid}/boundTagInfo
- Content-type: application/json
- Path Parameters:
    - uid: 欲綁定標的物之識別碼
- Request Body Example:
```json=
{
  "boundHeight": 0,
  "boundTagId": 44889,
  "boundTime": "2020-08-17T08:40:04.288Z"
}
````
- Request Parameters
    - boundHeight:綁定標籤之高度
    - boundTagId:綁定標籤之十進制識別碼
    - boundTime:綁定時間
    
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": {
    "itemUid": "A000001",
    "itemSn": 1,
    "boundSn": 2, (綁定序列號)
    "boundTagId": 44889,
    "boundTime": "2020-08-17T08:40:04.288Z",
    "boundHeight": 0
  }
}
```
#### 3.7 以Uid查詢特定標的物綁定狀態: GET /rest/1.0.0/items/{uid}/boundTagInfo
- Path Parameters:
    - uid: 欲查詢標的物之識別碼
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": {
    "itemUid": "A000001",
    "itemSn": 1,
    "boundSn": 2, (綁定序列號)
    "boundTagId": 44889,
    "boundTime": "2020-08-17T08:40:04.288Z",
    "boundHeight": 0
  }
}
```
#### 3.8 解除綁定標的物:PUT /rest/1.0.0/items/{uid}/boundTagInfo/clear
- Path Parameters:
    - uid: 欲解綁定標的物之識別碼
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": {
    "itemUid": "A000001",
    "itemSn": 1,
    "boundSn": null,
    "boundTagId": null,
    "boundTime": null,
    "boundHeight": null
  }
}
```

#### 3.9 查詢標的物分類: GET /rest/1.0.0/item/kinds
定位標的物之類型(設備、病患、護理人員)
- Query Parameters:
    - sort: [{"prop":"name", "dir":"asc"}] or empty
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": [
    {
      "sn": 2,
      "name": "EQUIPMENT",
      "displayName": "設備",
      "icon": null
    },
    {
      "sn": 1,
      "name": "STAFF",
      "displayName": "人員",
      "icon": null
    }
  ]
}
```

#### 3.10 查詢標的物綁定記錄: POST /rest/1.0.0/item/boundInfoHistory/query
- Content-type: application/json
- Request Body Example:
```json=
//完整格式
{
  "cond": {
    "boundStatus": true,
    "boundTagId": 0,
    "endTime": "2020-12-23T16:35:01.558Z",
    "itemDisplayNameCond": {
      "text": "string",
      "partial": true
    },
    "itemNameCond": {
      "text": "string",
      "partial": true
    },
    "itemUidCond": {
      "text": "string",
      "partial": true
    },
    "itemKindCond": {
      "text": "string",
      "partial": true
    },
    "startTime": "2020-12-23T16:35:01.558Z"
  },
  "paging": {
    "page": 0,
    "pageSize": 10
  },
  "sort": [
    {
      "prop": "property1",
      "dir": "ASC"
    }
  ]
}

//查詢範例
{
  "cond": {
    "endTime": "2020-12-23T16:35:01.558Z",
    "startTime": "2020-11-23T16:35:01.558Z"
  },
  "paging":null,
  "sort":[{"dir":"DESC","prop":"boundTime"}]
}
```
- Request Parameters
    - cond:配對查詢條件
    - cond.boundStatus:綁定條件，true為查詢目前仍為綁定狀態的記錄，false為查詢目前非為綁定狀態的記錄，不指定則查詢所有
    - cond.boundTagId:所查詢之標籤ID
    - cond.endTime: 查詢時間之結束時間(UTC)
    - cond.itemDisplayNameCond: 針對標的物顯示名稱進行過濾之條件物件
    - cond.itemDisplayNameCond.text:針對標的物顯示名稱進行搜尋之文字條件
    - cond.itemDisplayNameCond.partial: 針對標的物顯示名稱進行搜尋之搜尋方式。true:部分符合,false:完全符合
    - cond.itemNameCond: 針對標的物識別名稱進行過濾之條件物件
    - cond.itemNameCond.text:針對標的物識別名稱進行搜尋之文字條件
    - cond.itemNameCond.partial: 針對標的物識別名稱進行搜尋之搜尋方式。true:部分符合,false:完全符合
    - cond.itemNameCond: 針對標的物識別名稱進行過濾之條件物件
    - cond.itemNameCond.text:針對標的物識別名稱進行搜尋之文字條件
    - cond.itemNameCond.partial: 針對標的物識別名稱進行搜尋之搜尋方式。true:部分符合,false:完全符合
    - cond.itemKindCond: 針對標的物類型進行過濾之條件物件
    - cond.itemKindCond.text:針對標的物類型進行搜尋之文字條件
    - cond.itemKindCond.partial: 針對標的物類型進行搜尋之搜尋方式。true:部分符合,false:完全符合
    - cond.startTime: 查詢時間之起始時間(UTC)
    - cond.paging: 分頁設定物件
    - cond.paging.page: 所欲回傳之分頁頁數
    - cond.paging.pageSize: 每分頁設定回傳之資料筆數
    - cond.sort:排序設定物件
    - cond.sort.prop: 所欲排序之欄位名稱
    - cond.sort.dir: 排序方式, ASC: 遞增, DESC: 遞減
    
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": {
    "histories": [
      {
        "sn": 2,
        "itemUid": "P-00002",
        "itemSn": 2,
        "itemName": "Guest2",
        "itemDisplayName": "Guest 2",
        "boundTagId": 46624,
        "boundTime": "2020-09-13T06:05:00.04Z",
        "unboundTime": "2020-12-15T08:49:30.432Z"
      },
      {
        "sn": 4,
        "itemUid": "E-00002",
        "itemSn": 4,
        "itemName": "光源機_002",
        "itemDisplayName": "光源機 002",
        "boundTagId": 37527,
        "boundTime": "2020-09-13T06:11:38.351Z",
        "unboundTime": "2020-12-22T06:24:23.028Z"
      },
      (以下省略)
    ],
    "count": 5,
    "pageMeta": {
      "total": 5,
      "page": 0,
      "pageSize": -1,
      "totalPages": 1
    },
    "lastQueryTime": "2020-12-24T03:21:08.465Z"
  }
}
```

### 4. 功能API - TargetState 目標物狀態群組
查看目標物即時狀態之功能群組，可查詢標的物或標籤之坐標、告警狀態等訊息。
#### 4.1 批次查詢標的物狀態: POST /rest/1.0.0/targetState/batchQuery
- Content-type: application/json
- Request Body Example:
```json=
//完整格式
[
  {
    "targets": {
      "itemUids": [
        "string"
      ],
      "tagIds": [
        0
      ],
      "itemOnly": true,
      "bound": true,
      "unknown": true
    },
    "conds": {
      "status": {
        "spaceUid": 0,
        "spaceName": "string",
        "alarm": true,
        "state": "string",
        "online": true,
        "noPositionOnly": true,
        "offlineWithinTime": 0,
        "tagStateExists": true,
        "batLowerThan": 0
      },
      "itemInfo": {},
      "tagInfo": {}
    },
    "destEpsg": "string",
    "countOnly": true
  }
]
````
- Request Parameters
    - targets.itemUids: 所查詢之標的物識別碼，null或不輸入代表不指定
    - targets.tagIds: 所查詢之標籤識別碼，需由標籤ID轉為十進制，null或不輸入代表不指定
    - targets.itemOnly: 查詢結果是否限定於查詢標的物，若為true則僅回傳標的物資料，若為null或不輸入代表不指定，則回傳可為標的物或標籤
    - targets.bound: 查詢結果是否限定於綁定狀態，若true則為綁定，若為null或不輸入代表不指定
    - targets.unkown: 查詢結果是否過濾未註冊的標籤，若false則不回傳未註冊的標籤
    - cond.status.spaceUid: 查詢之空間識別碼過濾條件
    - cond.status.spaceName: 查詢之空間名稱過濾條件，可參照空間名稱對應表
    - cond.status.alarm: SOS告警狀態過濾條件，設為true則表示僅回傳目前為SOS狀態之結果，null或不輸入代表不指定
    - cond.status.state: 查詢對象之狀態，可參照狀態碼對應表，null或不輸入代表不指定
    - cond.status.noPositionOnly: 查詢對象之狀態，可參照狀態碼對應表，null或不輸入代表不指定
    - cond.status.offlineWithinTime: 於所設定之時間內為離線的對象，單位為秒
    - cond.status.tagStateExists: 標籤是否曾上線，若為true表示標籤曾經上線過，若為false則表示標籤未曾上線
    - cond.status.batLowerThan: 標籤低於之百分比過濾條件，輸入數字代表過濾電量之百分比
    - conds.itemInfo: 針對標的物屬性的過濾條件，目前為保留欄位不使用 
    - conds.tagInfo: 針對標籤設定的顧慮條件，目前為保留欄位不使用 
    - destEpsg: 回傳結果標的物之定位坐標系統,若為null則採用預設epsg:3826坐標系統
    - countOnly: 查詢結果是否僅回傳符合查詢條件之計數結果，若為true則為僅回傳計數結果

-  空間名稱(spaceName)對應表
   | 空間名稱       | 參數字串      |
    |----------------|---------------|
    | KSPH_1F|  KSPH_1F|
    
    
-  狀態碼對應表
    | 狀態名稱       | 參數字串      |
    |---------------|---------------|
    | 低電量         | $BAT_LOW      |
    | 異常靜止       | $ABN_REST     |
    | 超出範圍       | $CROSS_REGION |
:::info
本API Request Body為 json 陣列, 可於單次API給定多組查詢條件，一次性查詢
:::
- Response
    ```json=
    {
      "success": true,
      "msg": null,
      "data": [
        {
          "count": 0, （符合搜尋條件結果之個數)
          "targetStates": [
            {
              "type": "string", (目標類型: item/tag)
              "itemUid": "P-00001",（標的物識別碼)
              "tagId": 0, (標籤識別碼十進制)
              "unknown": true, (是否屬於未註冊標籤)
              "online": true, (是否上線)
              "states": [ (標的物所觸發之狀態)
                "$ABN_REST"
              ],
              "alarm": false, （標的物是否為SOS告警狀態)
              "lastTagState": { (上一筆標籤狀態資訊)
                "tagId": 44889, （標籤識別碼)
                "online": true, (是否上線)
                "position": {   (定位坐標)
                  "x": 170206.85137839013,
                  "y": 2542885.0478511434,
                  "z": 19.50000000059082
                },
                "updateTime": "2020-08-31T03:00:32.682Z", (定位更新時間)
                "infoTime": "2020-08-31T03:00:32.482Z", (資訊更新時間)
                "positionTime": "2020-08-31T03:00:32.682Z", (標籤資訊更新時間)
                "alarmBit": false, (標籤SOS位元狀態)
                "bat": 37, (標籤電量)
                "rest": true,（標籤是否為靜止狀態)
                "sleep": false,（標籤是否為休眠狀態)
                "spaceUid": 1, (標籤所屬最上層空間識別碼)
                "regionName": "GIPS", (標籤所屬區域名稱)
                "regionDisplayName": "GIPS", (標籤所屬區域顯示名稱)
                "spaceUidList": [ (標籤所屬空間識別碼)
                  1
                ],
                "moving": false (標籤是否為移動狀態)
              },
              "lastValidPos": { (最後有效定位點資訊)
                "spaceUid": 1, (最後有效定位點所屬空間識別碼)
                "position": {  (最後有效定位點位坐標)
                  "x": 170206.85137839013,
                  "y": 2542885.0478511434,
                  "z": 19.50000000059082
                },
                "positionTime": "2020-08-31T03:00:32.482Z",(最後有效定位時間)
                "tagId": 44889  (最後有效定位點所綁定之標籤識別碼)
              }
            }
          ],
          "lastQueryTime": "2020-08-31T03:00:32.73Z",
          "epsgCode": "epsg:3826" （回傳結果之坐標系統)
        }
      ]
    }
    ```

#### 4.2 解除標的物SOS告警: POST /rest/1.0.0/itemStates/{itemUid}/clearAlarm
- Content-type: application/json
- Path Parameters:
    - uid: 欲解除SOS告警之標的物識別碼
- Request Body Example:
```json=
///目前規格為標準產品之結構，彰基後續將依據訂定標的物UML擴充
{
  "boundTagId":"null"
}
````
- Request Parameters
    - boundTagId:解除標的物SOS告警所對應綁定之標籤識別碼，當不帶入時，則解除之SOS狀態為最新的告警事件
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": 0
}
```

### 5. 功能API - Space 群組
#### 5.1 取得專案所建立之Space: POST /rest/1.0.0/space/query
- Content-type: application/json
- Request Example
```json=
{
  "query": {},
  "paging": {
    "page": 0,
    "pageSize": 10
  },
  "sort": [
    {
      "prop": "name",
      "dir": "ASC"
    }
  ]
}
```
- Request Parameters:
    - query: 查詢條件，目前為空物件
    - paging: 分頁設定物件
    - paging.page: 所欲回傳之分頁頁數
    - paging.pageSize: 每分頁設定回傳之資料筆數
    - sort:排序設定物件
    - sort.prop: 所欲排序之欄位名稱
    - sort.dir: 排序方式, ASC: 遞增, DESC: 遞減
- Response Body
```json=
{
  "success": true,
  "msg": null,
  "data": {
    "total": 2,
    "page": 0,
    "pageSize": 10,
    "totalPages": 1,
    "content": [
      {
        "sn": 1, (空間流水號)
        "uid": 1, (空間識別碼)
        "groupUid": 1, (空間所屬群組識別碼)
        "name": "GIPS_1", (空間名稱)
        "groupName": "group_1", (空間所屬群組名稱)
        "displayName": "GIPS Office 1", (空間顯示名稱)
        "description": null, (空間描述說明)
        "epsgCode": "EPSG:3826", (空間是用座標系統)
        "boundingBox": { (空間外包矩形)
          "minX": 170192.34467216372,
          "minY": 2542873.9212778755,
          "maxX": 170216.31913301424,
          "maxY": 2542893.2526512295
        },
        "builtIn": true,
        "mapUploaded": true, (空間地圖資料是否上傳)
        "mapOsmPath": null,
        "ordinal": 1,
        "deleted": false
      },
      (以下省略)
    ],
    "count": 2
  }
}
```
