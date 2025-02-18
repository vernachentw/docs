---
title: GIPSense Map API User Manual (EN)
tags: [Product, EN]

---

# GIPSense Map API User Manual (EN)
[TOC]

## Version
| Updated Date | Notes |
|--------------|-------|
| 2023/01/29   | 1st Release|

## Introduction
GIPSense offers an Embedded Map Platform API to meet the needs of third-party applications to integrate location maps, and to provide fast development tools. The API can directly integrate the map interface into the user interface of the application system without having to be familiar with the map interface development framework. The embedded map platform is provided in an iframe format, offering developers the necessary map operation functions API for location application integration. It can be called through a channel communication method or directly controlled through URL GET parameters. The resource path of the embedded map platform provided by GIPSense is usually in the following format:
```http=
http://{FQDN or IP}:8686/rtls/app/{lang}/map
```
- FQDN or IP: The fully qualified domain name or IP address of the service
- lang: If the GIPSense is a multilingual version, the language version should be entered here, such as "zh-tw" or "en". If it is not a multilingual version, this path can be omitted.
The following describes two integration methods.

## Method-A. Communication and calling through channel
- message format: {type: string, cmd: string, data: object or string}
    - Parameters:
        - type: message type, candidates values: **request, response, info**
        - cmd: cmd name, refernce to the below command api
        - data: data, which will vary depending on the type and command name.
- There are two types of API provided: 
    - **Type A (Send Request and Get Response)**: The developer controls the map platform app by sending a request message, and the app will respond with a response message indicating that the command has been received. This can be used to control the app's behavior or retrieve the app's status.
    - **Type B**: The map platform app proactively sends a message to inform the developer that a certain action has been completed.
- Regarding authorization:
 Due to the sensitivity of map information, the GIPSense backend can restrict access to the map data. In this case, an API Key must be provided by the FAE (field application engineer) and authenticated through the **"configure"** function in Type A. Only then can the map data be displayed. 

### TypeA - Send Request and Get Response
#### 1.1 Confirming API readiness: isReady
- Format: isReady()
- Description: Confirm that the API is ready to start operations.
- Request Example:
    ```javascript=
    window.postMessage({type: "request", cmd: "isReady", data: null})
    ```
- Response Example:
    ```json=
    {type: "response", cmd: "isReady", data: true}
    ```

#### 1.2 Confirming floor map information is loaded: isSpaceReady

- Format: isSapceReady()
- Description: Confirm whether the current space's map information has been loaded.
- Request Example:
    ```javascript=
    window.postMessage({type: "request", cmd: "isSpaceReady", data: null})
    ```
- Response Example:
    ```json=
    {type: "response", cmd: "isSapceReady", data: true}
    ```
#### 1.3 Configuring Special Settings: configure

- Format:configure({key: string, value: unknown })
- Parameters:
    - key: string - The key value of the operation type to be executed
    - value: unknown - The value to be set for the operation type, the type depends on the configuration content
- Description:
    - Configure special settings, which controls the settings that affect the entire map platform.
    - Setting up developer authentication:
        - key: authentication
        - value:{apiKey: string, appId: string}。
- Request Example:
    - Setting up API token authentication information:
    ```json=
    window.postMessage({type: "request", cmd: "configure", data: {key: "authentication", value: {apiKey: "xxx", appId: "ooo"} }})
    ```
    :::info
    This operation does not require authentication in the test environment and the API key will be released in the production environment
    :::

#### 1.4 Setting the Desired Target Item: setTargetItemUids
- Format: setTargetItemUids(uids)
- Parameters:
    - uids: Array(String) - The array of monitored item uids of the target items to be displayed
    - Description: Only the specified personnel and points will be displayed on the map. Usually called after receiving the ready message.uids: Array(String)  - 欲顯示標的物的 monitored item uid 陣列
- Request Example:
    - All targets: 
    ```javascript=
    window.postMessage({type: "request", cmd: "setTargetItemUids", data: null})
    ```
    - Specifying two targets:
    ```javascript=
    window.postMessage({type: "request", cmd: "setTargetItemUids", data: ["P-00001", "P-00002"]})
    ```
- Response Example:
    ```json=
    {type: "response", cmd: "setTargetItemUids", data: ["P-00001", "P-00002"]}
    ```

#### 1.5 Obtaining the Desired Target Item: getTargetItemUids
- Format: getTargetItemUids()
- Description: Returns a uids array that gets the currently desired target itemUid.
- Request Example:
    ```javascript=
    window.postMessage({type: "request", cmd: "getTargetItemUids", data: null})
    ```
- Response Example:
    - All targets: 
    ```json=
    {type: "response", cmd: "getTargetItemUids", data: null}
    ```
    - Currently specifies two:
    ```json=
    {type: "response", cmd: "getTargetItemUids", data: ["P-00001", "P-00002"]}
    ```
    
#### 1.6 Setting the Desired Tag: setTargetTagIds

- Format: setTargetTagIds(ids)
- Parameters:
    - ids: Array(number) - The id(Decimal) array of the tag you want to display
- Description: The specified label, the location will be displayed on the map. Usually called after receiving the ready message.
- Request Example:
    - All tags: 
    ```javascript=
    window.postMessage({type: "request", cmd: "setTargetTagIds", data: null})
    ```
    - Specify two: 
    ```json=
    window.postMessage({type: "request", cmd: "setTargetTagIds", data: [44889, 46624]})
    ```
- Response Example:
    ```json=
    {type: "response", cmd: "setTargetTagIds", data: [44889, 46624]}
    ```

#### 1.7 Obtaining the Desired Tags: getTargetTagIds

- Format: getTargetTagIds()
- Description: Returns an ids array that gets the current desired tag id
- Request Example:
    ```javascript=
    window.postMessage({type: "request", cmd: "getTargetTagIds", data: null})
    ```
- Response Example:
    - All tags: 
    ```json=
    {type: "response", cmd: "getTargetTagIds", data: null}
    ```
    - Currently specifies two:
    ```json=
    {type: "response", cmd: "getTargetTagIds", data: [44889, 46624]}
    ```

#### 1.8 Set Whether to Show Only Item Locations: setItemOnly

- Format:setItemOnly(value: boolean)
- Parameters:
    - value: (boolean) - Determines whether to show only item locations
- Description: 設定是否只顯示標的物點位。
- Request Example:
    - Show only items:
    ```javascript=
    window.postMessage({type: "request", cmd: "setItemOnly", data: true})
    ```
    - Show both items and tags:
    ```javascript=
    window.postMessage({type: "request", cmd: "setItemOnly", data: false})
    ```
- Response Example:
    ```json=
    {type: "response", cmd: "setItemOnly", data: true}
    ```

#### 1.9 Get the Settings of Whether to Show Only Item Locations: getItemOnly

- Format:getItemOnly()
- Description:  Gets the current setting of whether to show only item locations
- Request Example:
    ```javascript=
    window.postMessage({type: "request", cmd: "getItemOnly", data: null})
    ```
- Response Example:
    ```json=
    {type: "response", cmd: "getItemOnly", data: true}
    ```
#### 1.10 Set Whether to Use Last Valid Position When Unable to Locate: setUseLastValidPosition

- Format:setUseLastValidPosition(value: boolean)
- Parameters:
    - value: (boolean) - Determines whether to use last valid position when unable to locate.
- Description:
    - Sets whether to use last valid position when unable to locate.
    - Offline locations can only be displayed as the last valid position. If set to false, offline locations will not be displayed.
- Request Example:
    - Use last valid position when unable to locate:
    ```javascript=
    window.postMessage({type: "request", cmd: "setUseLastValidPosition", data: true})
    ```
    - Do not display location when unable to locate:
    ```javascript=
    window.postMessage({type: "request", cmd: "setUseLastValidPosition", data: false})
    ```
- Response Example:
    ```json=
    {type: "response", cmd: "setUseLastValidPosition", data: true}
    ```

#### 1.11 Get the Settings of Whether to Use the Last Valid Position When Unable to Locate: getUseLastValidPosition

- Format:getUseLastValidPosition()
- Description: Get whether to use the last valid position when unable to locate.
- Request Example:
    ```javascript=
    window.postMessage({type: "request", cmd: "getUseLastValidPosition", data: null})
    ```
- Response Example:
    ```json=
    {type: "response", cmd: "getUseLastValidPosition", data: true}
    ```
#### 1.12  Set Map Zoom Size:: setZoom

- Format:setZoom(zoom: number)
- Parameters:
    - zoom: (number) - zoom level
- Description: Specify the map zoom level and zoom to a suitable display size
- Request Example:
    ```javascript=
    window.postMessage({type: "request", cmd: "setZoom", data: 22})
    ```
- Response Example:
  ```json=
  {type: "response", cmd: "setZoom", data: 22}
  ```

#### 1.13 Get Map Zoom Level: getZoom
- Format:getZoom()
- Description: Get the current map zoom level.
- Request Example:
    ```javascript=
    window.postMessage({type: "request", cmd: "getZoom", data: null})
    ```    
- Response Example:
    ```javascript=
    {type: "response", cmd: "getZoom", data: 22}
    ```

#### 1.14 Switch Space: changeSpaceAsync
- Format: changeSpaceAsync(spaceName: string)
- Parameters:
    - spaceName:(string) - Space Name
- Description: Switch to the specified space by space name, asynchronously loading the map information for that space
- Request Example:
    ```javascript=
    window.postMessage({type: "request", cmd: "changeSpaceAsync", data: "2F"})
    ```
- Response Example::
    ```json=
    {type: "response", cmd: "changeSpaceAsync", data: "2F"}
    ```
    
#### 1.15 Get Current Space Info: getCurrentSpace
- Format:getCurrentSpace()
- Description: Obtains the current space information, providing a name for identification of different space.
- Request Example:
    ```javascript=
    window.postMessage({type: "request", cmd: "getCurrentSpace", data: null})
    ```
- Response Example:
    ```json=
    {type: "response", cmd: "getCurrentSpace", data: {name: "2F"}}
    ```
#### 1.16 Locate a Specific Target: locateItem

- Format: locateItem({itemUid: string, setCenter:boolean, setCurrent:boolean})
- Parameters:
    - itemUid: (string) - target uid
    - setCenter: (boolean) - whether to move the map with the target at the center
    - setCurrent: (boolean) - whether to set it as the current location (display popup)
- Description: Used to locate the location of a specific MonitoredItem, switching spaces based on the space where the location is located and displaying the location.
- Request Example:
    ```javascript=
    window.postMessage({type: "request", cmd: "locateItem", data: {itemUid: "P-00001", setCenter: true, setCurrent: true}})
    ```
- Response Example::
    ```json=
    {type: "response", cmd: "locateItem", data: {itemUid: "P-00001", setCenter: true, setCurrent: true}}
    ```
### TypeB - Get Info
#### 2.1 API Ready Notification: ready

- Description:A notification will be automatically sent out when the API is ready to ensure that the app will not perform operations on the API too soon.
- info response example:
    ```json=
    {type: "info", cmd: "ready", data: true}
    ```

#### 2.2 Space Change Notification:spaceChanged

- Description: A notification will be automatically sent out when the space changes to ensure that the app can synchronize space information.
- info response example:
    ```json=
    {type: "info", cmd: "spaceChanged", data: {name: "2F"}}
    ```


#### 2.3 Space Ready Notification: spaceReady

- Description: After switching space, when the map layer information is loaded, a notification will be automatically issued to ensure that the app can synchronize the map layer information. Please note that if you switch floors quickly, only the map layer information for the last switch floor will be loaded.
- info response example:
    ```json=
    {type: "info", cmd: "spaceReady", data: {name: "2F"}}
    ```
    
#### 2.4 Item Clicked Notification: itemClicked

- Description: After clicking an item on the map, a white-background tooltip and blue object name will appear above the item. Then, by clicking the blue item name, a notification will be issued, which will contain the content of the item's attributes.
- info response example:
    ```json=
    {type: "info", cmd: "itemClicked", data: {"uid":"P-00001","kind":"MEMBER","name":"Guest1", ... }}
    ```

## Method-B. Using the HTTP GET interface to call
The parameters provided by the embedded map when calling through the HTTP GET interface are summarized as blow:

- Request URI
```htmlembedded=
http:// {IP or FQDN}:8686/rtls/app/{lan}/map
```
- Method: GET
- Request Header: accept application/json
- Parameters:
    - mode: map mode, accept values:
        - default: Default mode, switch using space code
        - locating: Tracking mode, track with itemUid, the space will automatically switched.
    - space: space name. This parameter is only effective when **mode = default**, and the received parameter is the space name set in GIPSense.
    - locatingItemUid: Tracking Target Identifie. This parameter is valid when **mode = locating**, accepting the Uid of the monitoreditem uid in GIPSense.

