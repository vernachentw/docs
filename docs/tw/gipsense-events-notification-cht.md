---
title: GIPSense事件推播說明(CHT)
tags: [Product, CHT]

---

# GIPSense事件推播說明(CHT)
## 升版歷程
| Updated Date | Notes |
|--------------|-------|
| 2022/05/19   | 初版   |
| 2023/02/10   | 新增基站異常事件   |
| 2023/10/13   | 新增呼叫($SOS)事件類型   |

## 前言
為滿足電子圍籬跨系統應用需求，GIPSense可針對電子圍籬事件發生時，將事件訊息推播至外部系統，由外部應用系統來設計處理事件對應之邏輯，如介接蜂鳴器告警、顯示資訊在門禁系統等，本文件說明介接之方式。

### 推播類型

- A.標的物電子圍籬事件
    - 越區 ($CROSS_REGION)
    - 異常靜止 ($ABN_REST)
    - 低電量 ($LOW_BAT)
    - 呼叫 ($SOS)
- B.基站事件
    - 基站離線 ($A_OFFLINE)
    - 基站異常 ($A_ANOMALOUS)

### 推播方式
由接收端提供POST API URL，GIPSesne於事件發生時，發送事件推播至該URL，並以標準事件JSON訊息格式提供


### 標準推播JSON格式內容

#### A. 標的物電子圍籬事件
- 格式範例
```JOSN=
{
  "itemUid": "uid111111",
  "itemName": "",
  "itemDisplayName": "",
  "tagId": 42090,
  "spaceName": "",
  "region": "",
  "eventName":"$CROSS_REGION", [codelist: $CROSS_REGION, $ABN_REST,$LOW_BAT]
  "eventDisplayName": "",
  "eventTime": "2021-01-01T00:00:00.000Z",
  "geofenceRuleName": ""
}
```
- 欄位說明
    - itemUid: 追蹤對象識別碼
    - itemName: 追蹤對象識別名稱
    - itemDisplayName: 追蹤對象顯示名稱
    - tagId: 追蹤對象所配對之定位標籤識別碼
    - spaceName: 電子圍籬事件所發生時之所在空間名稱
    - region: 電子圍籬事件所發生時之所在區域名稱
    - eventName: 電子圍籬事件識別名稱
    - eventDisplayName: 電子圍籬事件識別顯示名稱
    - eventTime: 電子圍籬事件觸發之時間點
    - geofenceRuleName: 電子圍籬規則名稱

#### B. 基站事件

- 格式範例
```JOSN=
{
  "eventName": "$A_OFFLINE",[codelist: $A_OFFLINE, $A_ANOMALOUS]
  "eventDisplayName": "基站離線",[codelist: 基站離線, 基站異常]
  "eventTime": "2023-02-03T04:16:21.206Z",
  "hexAnchorIds": [
    "1093",
    "10C5",
    "10C7",
    "1105",
    "1108",
    "1109" 
  ]
}
```
- 欄位說明
    - eventName: 基站事件識別名稱
    - eventDisplayName: 基站事件識別顯示名稱
    - eventTime: 基站事件觸發之時間點
    - hexAnchorIds: 發生異常之基站識別碼
