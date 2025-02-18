---
title: GIPSense Events Notification
tags: [Product, EN]

---

# GIPSense Events Notification 
## Change Log
| Updated Date | Notes |
|--------------|-------|
| 2023/08/01   | Initial Release   |
| 2023/10/13   | Add Call service($SOS) event type   |


## Introduction
To meet the requirements of cross-system applications for electronic geofences, GIPSense can push event messages to external systems when geofence events occur. External application systems can then design the corresponding event logic, such as integrating with buzzer alarms, displaying information on access control systems, etc. This document explains the methods of integration.

### Types of Notification

- A. Item Geofence Events
    - Cross Region ($CROSS_REGION)
    - Abnormal Rest ($ABN_REST)
    - Low Battery ($LOW_BAT)
    - Call Service ($SOS)
- B.Anchor Events
    - Anchor Offline  ($A_OFFLINE)
    - Anchor Anomalous  ($A_ANOMALOUS)

### Notification Method
The receiving end provides a POST API URL. When an event occurs, GIPSense sends the event push to that URL, using the standard event JSON message format.


### Standard Notification JSON Format Content

#### A. Item Geofence Events
- Format Example
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
- Field Explanation
    - itemUid: Identifier of the tracked object
    - itemName: Identifier name of the tracked object
    - itemDisplayName: Display name of the tracked object
    - tagId: Identifier of the positioning tag paired with the tracked object
    - spaceName: Name of the space where the geofence event occurred
    - region: Name of the region where the geofence event occurred
    - eventName: Identifier name of the geofence event
    - eventDisplayName: Display name of the geofence event
    - eventTime: Time point when the geofence event was triggered
    - geofenceRuleName: Name of the geofence rule

#### B. Anchor Events

- Format Example
```JOSN=
{
  "eventName": "$A_OFFLINE",[codelist: $A_OFFLINE, $A_ANOMALOUS]
  "eventDisplayName": "Anchor offline",[codelist: AnchorOffline, AnchorAbnormal]
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
- Field Explanation
    - eventName: Identification name of the anchor event
    - eventDisplayName: Display name of the anchor event
    - eventTime: Time point when the anchor event is triggered
    - hexAnchorIds: Identifiers of the anchors where the anomaly occurred
