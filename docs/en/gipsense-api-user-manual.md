---
title: GIPSense API User Manual (EN)
tags: [Product, EN]

---

# GIPSense API User Manual (EN)
[TOC]

## Version
| Updated Date | Notes |
|--------------|-------|
| 2023/01/29   | 1st Release   |
| 2023/11/28   | 1. Add Embedded API document link<br> 2. Add Notification Service document link <br> 3. Remove the content of Embedded API.|

## A. Resource Description:
:::success
This document provides an introduction to the main APIs of GIPSense, for complete API usage, please refer to the description in the corresponding Swagger document.
:::
- demoIP: uwbdemo1.gips.com.tw
- GIPSense System Architecture
![GIPSense_architechture](https://hackmd.io/_uploads/HJjKjJXr6.png)

### 1. GIPSense RTLS Platform
- URL: **http://{demoIP}:8686/rtls/app**
- Login: admin/!p@ssw0rd
- The GIPSense management platform provides essential features for the complete location scenario, including geo-fence management, black/white list, historical track, event alert, tag issuance control, and camera linkage.
### 2. GIPSense Swagger API
#### 2.1 Authentication API Swagger:
- URL: http://{demoIP}:8686/rtls/rest/auth/1.0.0/docs.html
- Provides specifications for authentication-related APIs.
- Must first authenticate to view **2.2** functional API Swagger page or perform functional API operations.
#### 2.2 GIPSense Functional API Swagger:
- URL: http://{demoIP}:8686/rtls/rest/1.0.0/docs.html
- Provides specifications for all GIPSense platform functional APIs.

### 3. GIPSense Embedded Map API
- URL: http://{demoIP}.tw:8686/rtls/app/zh-tw/map
- Embedded map is in the form of a webpage, and through the development of custom event functions, it provides developers with the ability to interact with the map through the window object in the DOM.
- Document Link: https://hackmd.io/@_x0Ms3tHS82GlQa3pRiqyw/rkYXyfVnj
### 4. GIPSense Notification Service
- To meet the requirements of cross-system applications for electronic geofences, GIPSense can push event messages to external systems when geofence events occur.
- Document Link: https://hackmd.io/@_x0Ms3tHS82GlQa3pRiqyw/rkplPmwn3

## B. API Usage Instructions:
### 1. Authentication API:
#### 1.1 Obtain Cookie through Auth API:
- POST /rest/auth/1.0.0/signin
    u: username
    p: password
- In programming, HttpCookieManager can be used to manage the Cookie values in the response and bring the SESSION value from the Cookie into subsequent functional API operations.

### 2. Functional API - TagInfo Group
Corresponding to the GIPSense management platform **Tag List** feature page, providing tag creation (register), editing, deletion, and other API operation capabilities.

#### 2.1 Batch query tags:POST /rest/1.0.0/tagInfo/query
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
  - paging: Pagination information, not specified means no pagination
  - paging.page: Page number, the first page number is 0, default is 0
  - paging.pageSize: Page size, 0 or negative value and not specified means no pagination
  - query: Search condition, not specified means search for all
  - query.bound: Whether the tag is in a bound state (true/false), not specified means no restriction
  - query.tagIds: Specifies the tag identification number, in decimal, can set multiple search groups
  - query.rfidIdCond.text: RFID value string
  - query.rfidIdCond.partial: Whether RFID is a full match or not.
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
        "displayName": "Test Tag",
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
#### 2.2 Query Tags by tag id:POST /rest/1.0.0/tagInfo/queryByTagIds
- Content-type: application/json
- Request Body Example:
```json=
[44889]
````
- Request Parameters
  - tagIds: An array of strings inputting the decimal identification codes of the tags to be queried.
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": [
    {
      "tagId": 44889,
      "height": 120,
      "displayName": "Test Tag",
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
### 3. Functional API - MonitoredItem Group
The API for MonitoredItem groups allows for the management of monitoring data such as healthcare workers and equipment through the creation, deletion, and binding/unbinding of tags on the GIPSense management platform.

#### 3.1 Batch query monitoredItem: POST /rest/1.0.0/item/query
- Content-type: application/json
- Request Body Example:
```json=
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
///Example
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
- Request Parameters:
  - paging: pagination information, not specified means no pagination
  - paging.page: page number, first page is 0, default is 0
  - paging.pageSize: page size, 0 or negative values and not specified means no pagination
  - query: query condition, not specified means all search
  - query.boundTagIds: query condition based on the identification code of the tags bound to the monitored item, not specified means no restriction
  - query.bound: whether the monitored item is in a bound state (true/false), not specified means no restriction
  - query.kinds: types of monitored items (personnel/equipment), not specified means no restriction
  - keywordCond: keyword search condition
  - keywordCond.text: keyword search text
  - keywordCond.partial: whether keyword search is partial match (true/false)
  - sort: search result sorting settings, null means no sorting set
  - sort.prop: sorting based on field
  - sort.dir: sorting direction (ASC/DESC)
  - sort.ignoreCase: whether sorting is case-sensitive
  - sort.nullHandling: handling mechanism for when sorting field is null value (NATIVE, NULLS_FIRST, NULLS_LAST)
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
        "name": "Operating Table-A000001",
        "displayName": "Operating Table",
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
#### 3.2 Query monitoredItem by Uid: POST /rest/1.0.0/item/queryByUids
- Content-type: application/json
- Request Body Example:
```json=
[
  "A000001"
]
````
- Request Parameters
    - tagIds: An array of strings inputting the decimal identification codes of the tags to be queried.
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
      "name": "Operating Table-A000001",
      "displayName": "Operating Table",
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
#### 3.3 Create MonitoredItem: POST /rest/1.0.0/items
- Content-type: application/json
- Request Body Example:
```json=
{
  "uid": "string",
  "kind": "string",
  "name": "string",
  "displayName": "string",
  "icon": "string",
  "phone": "string",
  "email": "string"
}
````
- Request Parameters
    - uid: Identifier of the object
    - kind: Type of the object
    - name: Name of the object
    - displayName: Display name of the object
    - icon: Icon displayed on the map for the object
    - phone: Phone property of the object
    - email: Email property of the object
    
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
        "name": "Operating Table-A000001",
        "displayName": "Operating Table",
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
#### 3.4 Update MonitoredItem: PUT /rest/1.0.0/items/{uid}
- Content-type: application/json
- Path Parameters:
    - uid: uid: Identification code of the item to be updated
- Request Body Example:
```json=
{
  "uid": "string",
  "kind": "string",
  "name": "string",
  "displayName": "string",
  "icon": "string",
  "phone": "string",
  "email": "string"
}
````
- Request Parameters
    - uid: Identifier of the object
    - kind: Type of the object
    - name: Name of the object
    - displayName: Display name of the object
    - icon: Icon displayed on the map for the object
    - phone: Phone property of the object
    - email: Email property of the object
    
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
        "name": "Operating Table-A000001",
        "displayName": "Operating Table",
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

#### 3.5 Delete MonitoredItem: DELETE /rest/1.0.0/items/{uid}
- Path Parameters:
    - uid: Identification code of the item to be deleted
- Response
```json=
{
  "success": true,
  "msg": "string",
  "data": {}
}
```
#### 3.6 Bind MonitoredItem: PUT /rest/1.0.0/items/{uid}/boundTagInfo
- Content-type: application/json
- Path Parameters:
    - uid: The identification code of the item to be bound
- Request Body Example:
```json=
{
  "boundHeight": 0,
  "boundTagId": 44889,
  "boundTime": "2020-08-17T08:40:04.288Z"
}
````
- Request Parameters
    - boundHeight: Height of the bound tag
    - boundTagId: Decimal identification code of the bound tag
    - boundTime: Binding time
    
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": {
    "itemUid": "A000001",
    "itemSn": 1,
    "boundSn": 2, (binding serial number)
    "boundTagId": 44889,
    "boundTime": "2020-08-17T08:40:04.288Z",
    "boundHeight": 0
  }
}
```
#### 3.7 Query specific Monitoreditem's binding status by Uid: GET /rest/1.0.0/items/{uid}/boundTagInfo
- Path Parameters:
    - uid: The identification code of the item to be queried
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": {
    "itemUid": "A000001",
    "itemSn": 1,
    "boundSn": 2, (binding serial number)
    "boundTagId": 44889,
    "boundTime": "2020-08-17T08:40:04.288Z",
    "boundHeight": 0
  }
}
```
#### 3.8 Unbind MonitoredItem: PUT /rest/1.0.0/items/{uid}/boundTagInfo/clear
- Path Parameters:
    - uid: The identification code of the item to be unbound
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

#### 3.9  Retrieve MonitoredItem Kind: GET /rest/1.0.0/item/kinds
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
      "displayName": "Equipment",
      "icon": null
    },
    {
      "sn": 1,
      "name": "STAFF",
      "displayName": "Staff",
      "icon": null
    }
  ]
}
```

#### 3.10 Query MonitoredItem Binding Records: POST /rest/1.0.0/item/boundInfoHistory/query
- Content-type: application/json
- Request Body Example:
```json=
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

//Example
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
    - cond: the overall query conditions
    - cond.boundStatus: filters records based on the binding status of the item (true for currently bound, false for currently unbound)
    - cond.boundTagId: filters records based on the ID of the bound tag
    - cond.endTime: the end time for the time range of the query (in UTC)
    - cond.itemDisplayNameCond: filters records based on the display name of the item
    - cond.itemDisplayNameCond.text: the text condition for searching the item display name
    - cond.itemDisplayNameCond.partial: the search mode for the item display name (true for partial match, false for exact match)
    - cond.itemNameCond: filters records based on the identification name of the item
    - cond.itemNameCond.text: the text condition for searching the item identification name
    - cond.itemNameCond.partial: the search mode for the item identification name (true for partial match, false for exact match)
    - cond.itemKindCond: filters records based on the type of the item
    - cond.itemKindCond.text: the text condition for searching the item type
    - cond.itemKindCond.partial: the search mode for the item type (true for partial match, false for exact match)
    - cond.startTime: the start time for the time range of the query (in UTC)
    - cond.paging: the pagination settings for the query
    - cond.paging.page: the page number to return
    - cond.paging.pageSize: the number of records to return per page
    - cond.sort: the sorting settings for the query
    - cond.sort.prop: the field to sort by
    - cond.sort.dir: the sorting direction (ASC for ascending, DESC for descending)
    
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
        "itemName": "Light_002",
        "itemDisplayName": "Light 002",
        "boundTagId": 37527,
        "boundTime": "2020-09-13T06:11:38.351Z",
        "unboundTime": "2020-12-22T06:24:23.028Z"
      },
      (and so on...)
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

### 4. Function API - TargetState Group
This is a group of functions that allows you to view the real-time status of targets, including information such as the coordinates of targets or tags and alarm status.
#### 4.1 Batch Query Target Status: POST /rest/1.0.0/targetState/batchQuery
- Content-type: application/json
- Request Body Example:
```json=
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
    - targets.itemUids: The identification code of the object being queried, null or not entered means not specified.
    - targets.tagIds: The tag identification code for the query, which needs to be converted from the tag ID to decimal, null or not input means not specified
    - targets.itemOnly: The query results are restricted to the queried item or not, if true, only the item data is returned, if null or not entered, it means not specified, and the return can be either the item or the label.
    - targets.bound: The query result is limited to the binding status, if true, it is binding, if null or not entered, it means unspecified.
    - targets.unkown: The query result specifies whether to filter out unregistered tags. If set to false, unregistered tags will not be included in the query results.
    - cond.status.spaceUid: Inquiry of filtering conditions for space identification code, reference to **API 5.1- Obtain the Spaces created within a project**
    - cond.status.spaceName: Inquiry of filtering conditions for space name, reference to **API 5.1- Obtain the Spaces created within a project**
    - cond.status.alarm: SOS alarm state filtering conditions, setting it as true means returning only the results that are currently in SOS state, null or not inputting means not specifying.
    - cond.status.state: Inquiry of the status of the target, reference to the status code correspondence table, null or not inputting represents not specifying.
    - cond.status.noPositionOnly: Inquiry of the positioning status of the object, if set to true, it means specifying to return results that are unable to be located, null or not inputting means not specifying.
    - cond.status.offlineWithinTime: Targets that are offline within the specified time, unit is seconds.
    - cond.status.tagStateExists: Whether the tag has been online before, if true, the tag has been online before, if false, the tag has never been online.
    - cond.status.batLowerThan: Filter conditions for tag battery percentage below a certain value, inputting a number represents the percentage of battery used for filtering.
    - conds.itemInfo: Filtering conditions for the attributes of the target object, currently this field is reserved and not in use. 
    - conds.tagInfo: Filtering conditions for the concerns set for the tags, currently this field is reserved and not in use. 
    - destEpsg: The coordinate system of the returned location of the object, if null, the default EPSG:3826 coordinate system will be used.
    - countOnly: Whether the query result only returns the count of the results that meet the query criteria, if true, it only returns the count result.
    
-  Target State Code 
    | State       | Parameter      |
    |----------------|---------------|
    | Low Battery         | $BAT_LOW      |
    | Abnormal stillness  | $ABN_REST     |
    
    :::info
    This API Request Body is a json array, which can give multiple sets of query conditions in a single API and query them at once.
    :::
- Response
    ```json=
    {
      "success": true,
      "msg": null,
      "data": [
        {
          "count": 0, （The number of results that match the search criteria.)
          "targetStates": [
            {
              "type": "string", (Target type: item/tag)
              "itemUid": "P-00001",（Identification Code of item)
              "tagId": 0, (The decimal identification code of the tag.)
              "unknown": true, (Whether it belongs to an unregistered tag)
              "online": true, (Whether the target is online or not)
              "states": [ (Item's triggered state)
                "$ABN_REST"
              ],
              "alarm": false, （The status of whether the object is in an SOS alarm state)
              "lastTagState": { ()
                "tagId": 44889, （Decimal tag ID)
                "online": true, (Whether the tag is currently online)
                "position": {   (The coordinates of the tag)
                  "x": 170206.85137839013,
                  "y": 2542885.0478511434,
                  "z": 19.50000000059082
                },
                "updateTime": "2020-08-31T03:00:32.682Z", (Location Update Time)
                "infoTime": "2020-08-31T03:00:32.482Z", (Information Update Time)
                "positionTime": "2020-08-31T03:00:32.682Z", (Tag Information Update Time)
                "alarmBit": false, (SOS Bit Status of Tag)
                "bat": 37, (Battery of Tag)
                "rest": true, (Tag Rest State)
                "sleep": false, (Tag Sleep State)
                "spaceUid": 1, (Top-level Space Identification Number of the Tag)
                "regionName": "GIPS", (Name of the Region the Tag Belongs to)
                "regionDisplayName": "GIPS", (Display Name of the Region the Tag Belongs to)
                "spaceUidList": [ (Space Identification Numbers of the Tag)
                1
                ],
                "moving": false (Tag Moving State)
                },
                "lastValidPos": { (Information of Last Valid Location Point)
                "spaceUid": 1, (Space Identification Number of Last Valid Location Point)
                "position": { (Coordinates of Last Valid Location Point)
                  "x": 170206.85137839013,
                  "y": 2542885.0478511434,
                  "z": 19.50000000059082
                },
                "positionTime": "2020-08-31T03:00:32.482Z",(The time of the last valid positioning)
                "tagId": 44889  (The id(decimal) of the tag bound to the last valid positioning point)
              }
            }
          ],
          "lastQueryTime": "2020-08-31T03:00:32.73Z",
          "epsgCode": "epsg:3826" （The coordinate system of the returned result)
        }
      ]
    }
    ```

#### 4.2 Clearing SOS Alarm of MonitoredItem: POST /rest/1.0.0/itemStates/{itemUid}/clearAlarm
- Content-type: application/json
- Path Parameters:
    - uid: Identifier of the item whose SOS alarm is to be cleared.
- Request Body Example:
```json=
{
  "boundTagId":"null"
}
````
- Request Parameters
    - boundTagId: The identification code of the tag corresponding to the cleared SOS alarm of the object. When not entered, the cleared SOS state is the latest alarm event.
- Response
```json=
{
  "success": true,
  "msg": null,
  "data": 0
}
```

### 5. Functional API - Space Group
#### 5.1  Obtain the Spaces created within a project: POST /rest/1.0.0/space/query
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
    - query: query condition, currently this field is reserved and not in use. 
    - paging: pagination setting object
    - paging.page: the desired page number for the returned data
    - paging.pageSize: the number of data records returned per page
    - sort: sorting setting object
    - sort.prop: the name of the column to sort by
    - sort.dir: sorting direction, ASC: ascending, DESC: descending
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
        "sn": 1, (Serial Number for Space)
        "uid": 1, (Idenfication code of space)
        "groupUid": 1, (Space Group Id)
        "name": "GIPS_1", (Space Name)
        "groupName": "group_1", (Space Group Name)
        "displayName": "GIPS Office 1", (Space Display Name)
        "description": null, (Space description)
        "epsgCode": "EPSG:3826", (Space coordinates system)
        "boundingBox": { (Space minimum bounding rectangle)
          "minX": 170192.34467216372,
          "minY": 2542873.9212778755,
          "maxX": 170216.31913301424,
          "maxY": 2542893.2526512295
        },
        "builtIn": true,
        "mapUploaded": true, (Whether the space map data is uploaded)
        "mapOsmPath": null,
        "ordinal": 1,
        "deleted": false
      },
      (and so on...)
    ],
    "count": 2
  }
}
```