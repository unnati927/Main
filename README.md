# Main
# STS Asset Panda Gateway

This project is an anti-corruption layer for integrating with Asset Panda. It adapts Asset Panda objects to STS's core
domain definitions and interface standards. STS's custom software should interact with this gateway, rather than
directly integrating with the Asset Panda API. This discipline promotes a cohesive system by providing a single
component that defines the mapping between Asset Panda and STS objects.

# Domain Distillation

STS Asset Panda Gateway is implemented the [Serverless Framework](https://serverless.com/). It is hosted using AWS
Lambda. It interfaces with other STS modules using a REST interface through AWS API Gateway. Client modules are
identified using an API key, allows misbehaving clients to be throttled by the API Gateway. This gateway interacts with
the Asset Panda API using a dedicated Asset Panda user account with broad privileges.

# Asset Panda Data Organization

Asset Panda provides an interface to a database which manages inventory of objects. STS uses Asset Panda to keep a record
of all tools used by every asset. In addition, Asset Panda also manages other data artifacts associated with tools
such as calibration documents associated equipment, location of equipment, owner of tool, and other relevant information.

All equipment are organized in five categories:

* Gauge (e.g. pressure gauge)
* Manual Wrench
* Hydraulic Wrench
* Mobile Device
* Pump
* Torque Gun

STS maintains 5 entities from which data can be fetched via asset panda API.

* asset (entity id: 73560)
* location (entity id: 73561)
* category (entity id: 73563)
* owner (entity id: 73564)
* part (entity id: 92471)

## Gauge

This Gateway defines objects that describe _gauge_. This is a gauge which transmits pressure information to TorqueFit
via a Bluetooth link. Gauge calibration must be certified, and the certification must be renewed at manufacturer
recommended intervals. The STS Asset Panda Gateway supports this requirement by identifying gauges and calibrations.

STS requires each gauge to be _commissioned_ at the site where it is used. To be commissioned, a wrench must be
registered with Asset Panda and be associated with a site.

Note: Pressure values are captured in PSI, unless otherwise stated.

## Manual Wrench

This Gateway defines objects that describe _manual torque wrenches_. These are hand-operated wrenches that
transmit tightening information to TorqueFit via a Bluetooth link. Torque wrench calibration must be certified, and the
certification must be renewed annually. The STS Asset Panda Gateway supports this
requirement by identifying wrenches and calibrations.

STS requires each wrench to be _commissioned_ at the site where it is used. To be commissioned, a wrench must be
registered with Asset Panda and be associated with a site.

Note: All torque values are captured in Newton-Meters (Nm).

## Hydraulic Wrench

This Gateway defines objects that describe _hydraulic torque wrenches_. These wrenches are driven by hydraulic pumps
which are electrically or pneumatically powered. Each pump is outfitted with a pressure gauge. Torque is correlated to
hydraulic or pneumatic pressure, which is transmitted to TorqueFit via a Bluetooth link. Torque wrench calibration must
be certified, and the certification must be renewed annually.

STS requires each wrench to be _commissioned_ at the site where it is used. To be commissioned, a wrench must be
registered with Asset Panda and be associated with a site.

Note: All torque values are captured in Newton-Meters (Nm).

## Mobile Device

This Gateway defines objects that describe _mobile device_. This is a computing device, such as tablet which
hosts TorqueFit application. It communicates with STS site server and with tools to collect tightening information.

STS needs to know keep an inventory of devices at each site where it is used. A mobile device must be registered with
Asset Panda and be associated with a site.


## Pump

This Gateway defines objects that describe _pump_. This is an electrically or pneumatically driven rotating machinery
which generates hydraulic energy required to tighten bolts to a high torque value (> 500Nm). Pumps are typically used
in conjunction with hydraulic wrench(s) and pressure gauge. Each wrench head has a pressure/torque chart which
is used to correlate pressure to torque, and requires calibration. However, not every pump requires a calibration certificate.

A typical pump has a pair of input/output port. These ports direct hydraulic fluid to and from the hydraulic wrench.
The cyclic flow of the fluid creates a ratcheting motion on the wrench head which in turn tightens the bolt. Some pumps
have up to 4 pairs of input/output ports.

### Smart Pump
Next generation pumps like the Hytorc Vector Pump have an on-board computer with a bluetooth radio, and integrated pressure
gauges. These pumps do not require an external pressure gauge, however, they require calibration to ensure accurate measurements
and performance.

## Torque Gun

This Gateway defines objects that describe _torque guns_. These are battery powered hand-operated motor driven wrenches that
transmit tightening information to TorqueFit via a Bluetooth link. Torque Gun calibration must be certified, and the
certification must be renewed periodically. The STS Asset Panda Gateway supports this requirement by identifying
Torque Gun wrenches and calibrations.

STS requires each torque gun to be _commissioned_ at the site where it is used. To be commissioned, a wrench must be
registered with Asset Panda and be associated with a site.

Note: All torque values are captured in Newton-Meters (Nm).

## Owner

Each asset belongs to an owner. URLs are keyed to the _owner_. The owner is a member of asset panda entity id 73564.
This corresponds to the site's name in asset panda. Example owners are: sji, bukom, elba, bechtel, qgc, stones, stw.

## Asset

The _asset_ models a physical item such as a gauge, manual wrench, hydraulic wrench and mobile device. Each asset is an
object in asset panda entity id 73560.

The snapshot is identified by `id`, while the entity is identified by `assetId`; the `id` is different for each distinct
snapshot, while the `assetId` is constant for all records that refer to the same physical object. Commissioning a manual
wrench asset is accomplished by posting a snapshot with relevant information filled in.

## Part

Each gauge, manual and hydraulic wrench part has a min/max range over which its calibration is valid. The most recently
created part snapshot with a given `partId` is _effective_. Parts are universal in STS; all owners have access to all
parts.

## Calibration

STS requires the time when the gauge, or wrench was calibrated, the time the calibration expires and the location of the
calibration certificate.

## Certificate

Certificates are PDF or image files with a printable calibration certificate. These files are stored by Asset Panda as
public objects in S3. The gateway will simply refer to the URL provided by Asset Panda. This avoids the need to cache or
process the files directly.

# Interface

The maximum length of an ID value is 2048 bytes. The maximum size of an asset, calibration or part message is 256KB.

The URL scheme is meant to allow access roles to use a prefix access policy. For example, Bukom users should have an
access policy permitting "/owner/bukom/\*".

## Assets

### Manual Wrench Assets

#### GET /owner/:owner/category/:category/asset

Get all the _effective_ asset snapshots from asset panda, where:

* `owner` is the name of the STS site (example `Bukom`).
* `category` is `Manual Wrench` for all manual torque wrenches.

```json
[
  {
    "id": "59c132d969702d82d8f60b00",
    "partId": "5b44ea965cdb80310f275dce",
    "modelName": "Tochnichi CEM3-G",
    "partNumber": "CEM200N3X19D-G-BTD",
    "serialNumber": "705374G",
    "bluetoothName": "ESD200v1.1.7-3105D2",
    "calibration": {
      "expiredAt": "2017-06-22T00:00:00.000Z",
      "calibratedAt": "2016-06-22T00:00:00.000Z",
      "certificate_url": ""
    },
    "firmwareRevision": null,
    "inspection": null
  },
  {
    "id": "59c132d969702d82d8f70b00",
    "partId": "5b44ea965cdb80310f275dce",
    "modelName": "Tochnichi CEM3-G",
    "partNumber": "CEM200N3X19D-G-BTD",
    "serialNumber": "705375G",
    "bluetoothName": "000190C3513B",
    "calibration": {
      "expiredAt": "2019-02-22T00:00:00.000Z",
      "calibratedAt": "2018-02-22T00:00:00.000Z",
      "certificate_url": ""
    },
    "firmwareRevision": null,
    "inspection": null
  },
  {
    "id": "59c132da69702d82d8fd0b00",
    "partId": "5b44eaa85cdb803133275e0c",
    "modelName": "Tochnichi CEM3-G",
    "partNumber": "CEM360N3X22D-G-BTD",
    "serialNumber": "707349G",
    "bluetoothName": "ESD200v1.1.7-310C4D",
    "calibration": {
      "expiredAt": "2017-10-05T00:00:00.000Z",
      "calibratedAt": "2016-10-05T00:00:00.000Z",
      "certificate_url": ""
    },
    "firmwareRevision": null,
    "inspection": null
  }
]
```

#### POST /owner/:owner/category/:category/asset

Creates a new asset snapshot in asset panda where:

* `owner` is the name of the STS site (example `Bukom`).
* `category` is `Manual Wrench` for all manual torque wrenches.

The body of the post request is as follows:

```json
  {
    "partId": "5b44ea965cdb80310f275dce",
    "serialNumber": "705374G",
    "bluetoothName": "ESD200v1.1.7-3105D2",
    "calibration": {
      "expiredAt": "2017-06-21T00:00:00.000Z",
      "calibratedAt": "2016-06-21T00:00:00.000Z"
    }
  }
```

The response data from the Gateway is as follows:
```
{
    "statusCode": 200,
    "body": {
        "id": "5c77019eb591ce2f1992e319",
        "partId": "5b44ea965cdb80310f275dce",
        "modelName": "Tohnichi CEM-BT",
        "partNumber": "CEM200N3X19D-G-BTD",
        "serialNumber": "705374G",
        "macAddress": null,
        "minimumTorque": 40,
        "maximumTorque": 200,
        "bluetoothName": "ESD200v1.1.7-3105D2",
        "calibration": {
            "expiredAt": "2017-06-21T00:00:00.000Z",
            "calibratedAt": "2016-06-21T00:00:00.000Z",
            "certificate_url": ""
        },
        "firmwareRevision": null,
        "inspection": null,
    },
    "headers": {
        "content-type": "application/json",
        "sts-media-type": "sts.v1"
    }
}
```

#### PUT /owner/:owner/category/:category/asset/:id

Edits/modifies a snapshot in asset panda as long as the _id_ is valid.

The body of the post request is as follows:

```json
  {
    "partId": "5b44ea965cdb80310f275dce",
    "serialNumber": "705374G",
    "bluetoothName": "ESD200v1.1.7-3105D2",
    "calibration": {
      "expiredAt": "2019-06-21T00:00:00.000Z",
      "calibratedAt": "2018-06-21T00:00:00.000Z"
    }
  }
```

The response data from the Gateway is as follows:
```
{
    "statusCode": 200,
    "body": {
        "id": "5c77019eb591ce2f1992e319",
        "partId": "5b44ea965cdb80310f275dce",
        "modelName": "Tohnichi CEM-BT",
        "partNumber": "CEM200N3X19D-G-BTD",
        "serialNumber": "705374G",
        "macAddress": null,
        "minimumTorque": 40,
        "maximumTorque": 200,
        "bluetoothName": "ESD200v1.1.7-3105D2",
        "calibration": {
            "expiredAt": "2019-06-21T00:00:00.000Z",
            "calibratedAt": "2018-06-21T00:00:00.000Z",
            "certificate_url": ""
        },
        "firmwareRevision": null,
        "inspection": null,
    },
    "headers": {
        "content-type": "application/json",
        "sts-media-type": "sts.v1"
    }
}
```

### Hydraulic/Pneumatic Wrench Assets

#### GET /owner/:owner/category/:category/asset

Get all the _effective_ asset snapshots from asset panda, where:

* `owner` is the name of the STS site (example `Bukom`).
* `category` is `Hydraulic Wrench` for all pump operated wrench heads.


```json
[
  {
    "id": "59c132d969702d82d8f60b00",
    "partId": "5b44ea965cdb80310f275dce",
    "modelName": "Hytorc Stealth 2",
    "partNumber": "1234ABC",
    "serialNumber": "705374G",
    "calibration": {
      "expiredAt": "2017-06-22T00:00:00.000Z",
      "calibratedAt": "2016-06-22T00:00:00.000Z",
      "certificate_url": ""
    },
    "firmwareRevision": null,
    "inspection": null
  },
  {
    "id": "59c132d969702d82d8f70b00",
    "partId": "5b44ea965cdb80310f275dce",
    "modelName": "Hytorc Stealth 4",
    "partNumber": "5678DEF",
    "serialNumber": "705375G",
    "calibration": {
      "expiredAt": "2019-02-22T00:00:00.000Z",
      "calibratedAt": "2018-02-22T00:00:00.000Z",
      "certificate_url": ""
    },
    "firmwareRevision": null,
    "inspection": null
  },
  {
    "id": "59c132da69702d82d8fd0b00",
    "partId": "5b44eaa85cdb803133275e0c",
    "modelName": "Hytorc Stealth 8",
    "partNumber": "9012GHI",
    "serialNumber": "707349G",
    "calibration": {
      "expiredAt": "2017-10-05T00:00:00.000Z",
      "calibratedAt": "2016-10-05T00:00:00.000Z",
      "certificate_url": ""
    },
    "firmwareRevision": null,
    "inspection": null
  }
]
```

#### POST /owner/:owner/category/:category/asset

Creates a new asset snapshot in asset panda where:

* `owner` is the name of the STS site (example `Bukom`).
* `category` is `Hydraulic Wrench` for all pump operated wrench heads.

The body of the post request is as follows:

```json
  {
    "partId": "5b44ea965cdb80310f275dce",
    "serialNumber": "705374G",
    "calibration": {
      "expiredAt": "2017-06-22T00:00:00.000Z",
      "calibratedAt": "2016-06-22T00:00:00.000Z"
    }
  }
```


The response data from the Gateway is as follows:

```json
{
    "statusCode": 200,
    "body": {
        "id": "5b44ea965cdb80310f275dce",
        "partId": "5b44ea965cdb80310f275dce",
        "serialNumber": "705374G",
        "modelName": "Hytorc Stealth 2",
        "partNumber": "STEALTH 2",
        "calibration": {
          "expiredAt": "2019-06-20T00:00:00.000Z",
          "calibratedAt": "2018-06-20T00:00:00.000Z",
          "certificate_url": ""
        },
        "firmwareRevision": "",
        "inspection": ""
    },
  "headers": {
      "content-type": "application/json",
      "sts-media-type": "sts.v1"
  }
}
```

#### PUT /owner/:owner/category/:category/asset/:id

Edits/modifies a snapshot in asset panda as long as the _id_ is valid.

The body of the post request is as follows:

```json
  {
    "partId": "5b44ea965cdb80310f275dce",
    "serialNumber": "705374G",
    "calibration": {
      "expiredAt": "2020-04-20T00:00:00.000Z",
      "calibratedAt": "2019-04-20T00:00:00.000Z"
    }
  }
```

The response data from the Gateway is as follows:

```json
{
    "statusCode": 200,
    "body": {
        "id": "5c783daf90d7b7715a869fe0",
        "partId": "5b44ea965cdb80310f275dce",
        "serialNumber": "705374G",
        "modelName": "Hytorc Stealth 2",
        "partNumber": "STEALTH 2",
        "calibration": {
          "expiredAt": "2020-04-20T00:00:00.000Z",
          "calibratedAt": "2019-04-20T00:00:00.000Z",
          "certificate_url": ""
        },
        "firmwareRevision": "",
        "inspection": ""
    },
  "headers": {
      "content-type": "application/json",
      "sts-media-type": "sts.v1"
  }
}
```

### Gauge Assets

#### GET /owner/:owner/category/:category/asset

Get all the _effective_ asset snapshots from asset panda, where:

* `owner` is the name of the STS site (example `Bukom`).
* `category` is `Gauge` for all gauges.

```json
[
  {
    "id": "5b731db5c1b56984497e02e4",
    "partId": "5b7c5796271c663e850db85d",
    "modelName": "CPG1500-ST-Z-S-PG569-NBTZ-53-W",
    "partNumber": "CPG1500-ST-Z-S-PG569-NBTZ",
    "serialNumber": "1A00RBR5CIM",
    "bluetoothName": "RNTFIGURE",
    "calibration": {
      "expiredAt": "2019-06-20T00:00:00.000Z",
      "calibratedAt": "2018-06-20T00:00:00.000Z",
      "certificate_url": ""
    },
    "firmwareRevision": "01.01.000",
    "inspection": null
  },
  {
    "id": "5b7326995cdb8056f834a833",
    "partId": "5b7c5796271c663e850db85d",
    "modelName": "CPG1500-ST-Z-S-PG569-NBTZ-53-W",
    "partNumber": "CPG1500-ST-Z-S-PG569-NBTZ",
    "serialNumber": "1A00RBR6MIX",
    "bluetoothName": "RNBNONE",
    "calibration": {
      "expiredAt": "2019-06-20T00:00:00.000Z",
      "calibratedAt": "2018-06-20T00:00:00.000Z",
      "certificate_url": ""
    },
    "firmwareRevision": "01.01.000",
    "inspection": null
  },
  {
    "id": "5b759d46271c664c98aa99eb",
    "partId": "5b7c5796271c663e850db85d",
    "modelName": "CPG1500-ST-Z-S-PG569-NBTZ-53-W",
    "partNumber": "CPG1500-ST-Z-S-PG569-NBTZ",
    "serialNumber": "1A00NB12M1Y",
    "bluetoothName": "RNTBD",
    "calibration": {
      "expiredAt": "2019-01-11T00:00:00.000Z",
      "calibratedAt": "2018-01-11T00:00:00.000Z",
      "certificate_url": ""
    },
    "firmwareRevision": "01.01.000",
    "inspection": null
  }
]
```

#### POST /owner/:owner/category/:category/asset

Creates a new asset snapshot in asset panda where:

* `owner` is the name of the STS site (example `Bukom`).
* `category` is `Gauge` for pressure gauges.

The body of the post request is as follows:

```json
  {
    "partId": "5b7c5796271c663e850db85d",
    "serialNumber": "1A00RBR5CIM",
    "calibration": {
      "expiredAt": "2019-06-20T00:00:00.000Z",
      "calibratedAt": "2018-06-20T00:00:00.000Z"
    }
  }
```

The response data from the Gateway is as follows:

```json
{
    "statusCode": 200,
    "body": {
        "id": "5c783daf90d7b7715a869fe0",
        "partId": "5b7c5796271c663e850db85d",
        "serialNumber": "1A00RBR5CIM",
        "modelName": "Wika CPG1500",
        "partNumber": "CPG1500VXXX",
        "bluetoothName": "",
        "calibration": {
          "expiredAt": "2019-06-20T00:00:00.000Z",
          "calibratedAt": "2018-06-20T00:00:00.000Z",
          "certificate_url": ""
        },
        "firmwareRevision": "",
        "inspection": ""
    },
  "headers": {
      "content-type": "application/json",
      "sts-media-type": "sts.v1"
  }
}
```

#### PUT /owner/:owner/category/:category/asset/:id

Edits/modifies a snapshot in asset panda as long as the _id_ is valid.

The body of the post request is as follows:

```json
  {
    "partId": "5b7c5796271c663e850db85d",
    "serialNumber": "1A00RBR5CIM",
    "bluetoothName": "BLE-ID",
    "calibration": {
      "expiredAt": "2019-06-20T00:00:00.000Z",
      "calibratedAt": "2018-06-20T00:00:00.000Z"
    }
  }
```

The response data from the Gateway is as follows:

```json
{
    "statusCode": 200,
    "body": {
        "id": "5c783daf90d7b7715a869fe0",
        "partId": "5b7c5796271c663e850db85d",
        "serialNumber": "1A00RBR5CIM",
        "modelName": "Wika CPG1500",
        "partNumber": "CPG1500VXXX",
        "bluetoothName": "BLE-ID",
        "calibration": {
          "expiredAt": "2019-06-20T00:00:00.000Z",
          "calibratedAt": "2018-06-20T00:00:00.000Z",
          "certificate_url": ""
        },
        "firmwareRevision": "",
        "inspection": ""
    },
  "headers": {
      "content-type": "application/json",
      "sts-media-type": "sts.v1"
  }
}
```

### Mobile Device Assets

#### GET /owner/:owner/category/:category/asset

Get all the _effective_ asset snapshots from asset panda, where:

* `owner` is the name of the STS site (example `Bukom`).
* `category` is `Mobile Device` for all mobile devices.

```json
[
  {
    "id": "59c132d669702d82d8e10b00",
    "partId": "5b44f34bc1b56987b5aa05f3",
    "modelName": "Aegex 10",
    "partNumber": "AEG-SKU-A10i6-073",
    "serialNumber": "BN60700199",
    "macAddress": "80-D2-1D-7B-BB-17"
  },
  {
    "id": "5b439ced90d7b729bda9f9f9",
    "partId": "5b44ea805cdb803114275dd4",
    "modelName": "AEGEX 10",
    "partNumber": "AEG-SKU-A10i7-433",
    "serialNumber": "BT71502030",
    "macAddress": "80:D2:1D:90:53:C3"
  },
  {
    "id": "5b439d9211e9b80d74abe252",
    "partId": "5b44ea805cdb803114275dd4",
    "modelName": "AEGEX 10",
    "partNumber": "AEG-SKU-A10i7-433",
    "serialNumber": "BT71502357",
    "macAddress": "80:D2:1D:90:50:7F"
  }
]
```

### Pump Assets

#### GET /owner/:owner/category/:category/asset

Get all the _effective_ asset snapshots from asset panda, where:

* `owner` is the name of the STS site (example `Bukom`).
* `category` is `Pump` for all pumps.

```json
[
  {
    "id": "5b731db5c1b56984497e02e4",
    "partId": "5b7c5796271c663e850db85d",
    "modelName": "Hytorc Vector",
    "partNumber": "Vector",
    "serialNumber": "185247935",
    "calibration": {
      "expiredAt": "2019-06-20T00:00:00.000Z",
      "calibratedAt": "2018-06-20T00:00:00.000Z",
      "certificate_url": ""
    },
    "firmwareRevision": "01.01.000",
    "inspection": null
  },
  {
    "id": "5b7326995cdb8056f834a833",
    "partId": "5b7c5796271c663e850db85d",
    "modelName": "Hytorc HY-115",
    "partNumber": "JETPRO 9.3",
    "serialNumber": "98521473",
    "calibration": null,
    "firmwareRevision": null,
    "inspection": null
  }
]
```

#### POST /owner/:owner/category/:category/asset

Creates a new asset snapshot in asset panda where:

* `owner` is the name of the STS site (example `Bukom`).
* `category` is `Pump` for all pumps (smart and non-smart).

The body of the post request is as follows:

```json
  {
    "partId": "5b7c5796271c663e850db85d",
    "serialNumber": "185247935",
    "calibration": {
      "expiredAt": "2019-06-20T00:00:00.000Z",
      "calibratedAt": "2018-06-20T00:00:00.000Z"
    }
  }
```

For non-smart pumps, data is as follows:

```json
 {
    "partId": "5b7c5796271c663e850db85d",
    "serialNumber": "98521473"
 }
```

The response data from the Gateway is as follows:

```json
{
    "statusCode": 200,
    "body": {
        "id": "5c783daf90d7b7715a869fe0",
        "partId": "5b7c5796271c663e850db85d",
        "serialNumber": "185247935",
        "modelName": "Hytorc Vector",
        "partNumber": "Vector 115V",
        "bluetoothName": "Vector-185247935",
        "calibration": {
          "expiredAt": "2019-06-20T00:00:00.000Z",
          "calibratedAt": "2018-06-20T00:00:00.000Z",
          "certificate_url": ""
        },
        "firmwareRevision": "",
        "inspection": ""
    },
  "headers": {
      "content-type": "application/json",
      "sts-media-type": "sts.v1"
  }
}
```

#### PUT /owner/:owner/category/:category/asset/:id

Edits/modifies a snapshot in asset panda as long as the _id_ is valid.

The body of the post request is as follows:

```json
  {
    "id": "5c783daf90d7b7715a869fe0",
    "partId": "5b7c5796271c663e850db85d",
    "serialNumber": "NewSerial",
    "calibration": {
      "expiredAt": "2019-06-20T00:00:00.000Z",
      "calibratedAt": "2018-06-20T00:00:00.000Z"
    }
  }
```

The response data from the Gateway is as follows:

```json
{
    "statusCode": 200,
    "body": {
        "id": "5c783daf90d7b7715a869fe0",
        "partId": "5b7c5796271c663e850db85d",
        "serialNumber": "NewSerial",
        "modelName": "Hytorc Vector",
        "partNumber": "Vector 115V",
        "bluetoothName": "Vector-185247935",
        "calibration": {
          "expiredAt": "2019-06-20T00:00:00.000Z",
          "calibratedAt": "2018-06-20T00:00:00.000Z",
          "certificate_url": ""
        },
        "firmwareRevision": "",
        "inspection": ""
    },
  "headers": {
      "content-type": "application/json",
      "sts-media-type": "sts.v1"
  }
}
```

### Torque Gun Assets

#### GET /owner/:owner/category/:category/asset

Get all the _effective_ asset snapshots from asset panda, where:

* `owner` is the name of the STS site (example `Bukom`).
* `category` is `Torque Gun` for all battery operated Torque Guns.

```json
[
  {
    "id": "59c132d969702d82d8f60b00",
    "partId": "5b44ea965cdb80310f275dce",
    "modelName": "Hytorc LION",
    "partNumber": "LION",
    "serialNumber": "1254789",
    "bluetoothName": "LI-85478",
    "calibration": {
      "expiredAt": "2017-06-22T00:00:00.000Z",
      "calibratedAt": "2016-06-22T00:00:00.000Z",
      "certificate_url": ""
    },
    "firmwareRevision": null,
    "inspection": null
  },
  {
    "id": "59c132d969702d82d8f70b00",
    "partId": "5b44ea965cdb80310f275dce",
    "modelName": "Hytorc Lithium",
    "partNumber": "Lithium II",
    "serialNumber": "521478",
    "bluetoothName": "LI-521472",
    "calibration": {
      "expiredAt": "2019-02-22T00:00:00.000Z",
      "calibratedAt": "2018-02-22T00:00:00.000Z",
      "certificate_url": ""
    },
    "firmwareRevision": null,
    "inspection": null
  }
]
```


#### POST /owner/:owner/category/:category/asset

Creates a new asset snapshot in asset panda where:

* `owner` is the name of the STS site (example `Bukom`).
* `category` is `Torque Gun` for all battery operated motor driven torque wrenches.

The body of the post request is as follows:

```json
  {
    "partId": "5b44ea965cdb80310f275dce",
    "serialNumber": "1254789",
    "bluetoothName": "LI-85478",
    "calibration": {
      "expiredAt": "2017-06-22T00:00:00.000Z",
      "calibratedAt": "2016-06-22T00:00:00.000Z"
    }
  }
```

The response data from the Gateway is as follows:
```
{
    "statusCode": 200,
    "body": {
        "id": "59c132d969702d82d8f60b00",
        "partId": "5b44ea965cdb80310f275dce",
        "modelName": "Hytorc LION",
        "partNumber": "LION",
        "serialNumber": "1254789",
        "macAddress": null,
        "minimumTorque": 40,
        "maximumTorque": 200,
        "bluetoothName": "LI-85478",
        "calibration": {
            "expiredAt": "2017-06-22T00:00:00.000Z",
            "calibratedAt": "2016-06-22T00:00:00.000Z"
            "certificate_url": ""
        },
        "firmwareRevision": null,
        "inspection": null
     },
    "headers": {
        "content-type": "application/json",
        "sts-media-type": "sts.v1"
    }
}
```


#### PUT /owner/:owner/category/:category/asset/:id

Edits/modifies a snapshot in asset panda as long as the _id_ is valid.

```json
  {
    "id": "59c132d969702d82d8f60b00",
    "partId": "5b44ea965cdb80310f275dce",
    "serialNumber": "UpdatedSerialNumber",
    "bluetoothName": "LI-85478",
    "calibration": {
      "expiredAt": "2017-06-22T00:00:00.000Z",
      "calibratedAt": "2016-06-22T00:00:00.000Z"
    }
  }
```

The response data from the Gateway is as follows:
```
{
    "statusCode": 200,
    "body": {
        "id": "59c132d969702d82d8f60b00",
        "partId": "5b44ea965cdb80310f275dce",
        "modelName": "Hytorc LION",
        "partNumber": "LION",
        "serialNumber": "UpdatedSerialNumber",
        "macAddress": null,
        "minimumTorque": 40,
        "maximumTorque": 200,
        "bluetoothName": "LI-85478",
        "calibration": {
            "expiredAt": "2017-06-22T00:00:00.000Z",
            "calibratedAt": "2016-06-22T00:00:00.000Z"
            "certificate_url": ""
        },
        "firmwareRevision": null,
        "inspection": null
    },
    "headers": {
        "content-type": "application/json",
        "sts-media-type": "sts.v1"
    }
}
```

## Get Tool Snapshots

### GET /tool/:id

An asset snapshot can be retrieved using the `id`.

```json
{
    "id": "5b731db5c1b56984497e02e4",
    "partId": "5b7c5796271c663e850db85d",
    "modelName": "CPG1500-ST-Z-S-PG569-NBTZ-53-W",
    "partNumber": "CPG1500-10000-0P05",
    "serialNumber": "1A00RBR5CIM",
    "maximumPressure": 10000,
    "bluetoothName": "CPG1500-1A00RBR5CIM",
    "calibration": {
        "expiredAt": "2019-06-20T00:00:00.000Z",
        "calibratedAt": "1970-01-01T00:00:00.000Z",
        "certificate_url": ""
    },
    "firmwareRevision": "01.01.000",
    "inspection": null
}
```

## Delete Snapshots

### DELETE /owner/:owner/category/:category/assets/:id

Asset snapshots are deleted from asset panda


```json
[
    {
      "id": "956846",
      "type": "Document",
      "name": "Wika_CPG1500_SN-1A00NB12M1Y.pdf",
      "created_at": "2018-08-16T15:50:47.000Z",
      "share_url": "https://login.assetpanda.com/d/3db0e2f3fc1615a1a1e9c343b6b66394",
      "url":
        "https://panda-assets-live.s3.amazonaws.com/uploads/document/document/956846/9a30666680d9ad137299ef52c68e5d78.pdf",
      "tags": []
    },
    {
      "id": "634214",
      "type": "Document",
      "name": "Calibration-SN-0118800942-20180122.pdf",
      "created_at": "2018-02-07T17:06:22.000Z",
      "share_url": "https://login.assetpanda.com/d/ebe018a12e59c3158a0ce85c97c65d32",
      "url":
        "https://panda-assets-live.s3.amazonaws.com/uploads/document/document/634214/54eaef25e19ced276e2d6d89fd6bd842.pdf",
      "tags": []
    }
]
```

## Tool Calibrations

### GET /calibration?id=<snapshotId>

Get all the calibration documents associated with an asset by `id`.


## Tool Parts (Models)

Part numbers are common across the STS system, they do not depend on the owner. Therefore there is no owner segment in
their URL scheme. Part numbers are associated with the model of the tools being used.


### GET /part/:partId

Get a given snapshot of a part, where :part = partId

```json
{
  "partId": "5b44eac95cdb8030fd275eba",
  "modelName": "Snap-on CTECH3",
  "partNumber": "CTECH3XN400X",
  "minimumTorque": 20,
  "maximumTorque": 400
}
```

### GET /part/category/:category

Gets all the part models that belong to a particular `category` (Example: Gauge).

```json
{
    "statusCode": 200,
    "body": [
      {
        "partId": "5b7c5796271c663e850db85d",
        "modelName": "WIKA CPG1500",
        "partNumber": "CPG1500-ST-Z-S-PG569-NBTZ",
        "maximumPressure": 10000
      },
      {
        "partId": "5b44eac95cdb8030fd275eba",
        "modelName": "Snap-on CTECH3",
        "partNumber": "CTECH3XN400X",
        "minimumTorque": 20,
        "maximumTorque": 400
      }
    ],
    "headers": {
      "content-type": "application/json",
      "sts-media-type": "sts.v1"
    }
}
```

### GET /part/parts

Get all effective snapshots of all parts. Note: the properties for parts belonging to each category is different.
For example, the key/value pairs for Hydraulic Wrenches do not exactly match Gauges.
Hydraulic Wrenches have the associated Pressure/Torque Curves. Pressure values in PSI, while Torque values are in
Newton-Meters (Nm).

```json
[
  {
    "partId": "5b44f34bc1b56987b5aa05f3",
    "modelName": "Aegex 10",
    "partNumber": "AEG-SKU-A10i6-073"
  },
  {
    "partId": "5b803e7f271c664cc79040d0",
    "modelName": "Advantech AIM-65AT",
    "partNumber": "AIM-65AT-22303000"
  },
  {
    "partId": "5b44ea965cdb80310f275dce",
    "modelName": "Tohnichi CEM-BT",
    "partNumber": "CEM200N3X19D-G-BTD",
    "minimumTorque": 40,
    "maximumTorque": 200
  },
  {
    "partId": "5b7c5796271c663e850db85d",
    "modelName": "WIKA CPG1500",
    "partNumber": "CPG1500-ST-Z-S-PG569-NBTZ",
    "maximumPressure": 10000
  },
  {
    "partId": "5b44eac95cdb8030fd275eba",
    "modelName": "Snap-on CTECH3",
    "partNumber": "CTECH3XN400X",
    "minimumTorque": 20,
    "maximumTorque": 400
  },
  {
    "partId": "5b92eec5271c667808d9ee9c",
    "modelName": "Hytorc Stealth 2",
    "partNumber": "STEALTH 2",
    "minimumTorque": 377,
    "maximumTorque": 2534,
    "pressureTorqueCurve": [
      {
        "P": 1500,
        "T": 377
      },
      {
        "P": 1600,
        "T": 403
      }
    ]
  },
   {
     "partId": "5c7583c9c1b56962ee8e95e1",
     "modelName": "Hytorc Vector",
     "partNumber": "Vector"
   }
]
```


## Commission Assets

Assets have to be commissioned for use in each site. This involves getting all assets in from asset panda and
pushing them to the site-specific database.

This is triggered by an SNS message structured as follows:
```
  Message: JSON.stringify({
    owner: "Cumulus",
    category: "Manual Wrench",
    site: "https://sts-test.azurewebsites.net"
  })
```
or
```
{"owner": "SMDS", "category": "Manual Wrench", "site": "https://sts-test.azurewebsites.net"}
```

The SNS message arn is exported as `AssetPandaGatewayTopicArn`. It can be referenced as follows: ` "Fn::ImportValue": "ParameterReaderRole"
AssetPandaGatewayTopicArn`. The SNS message has to be sent to the arn: `AssetPandaGatewayTopicArn`.
Message attribute should be as follows:
```
  MessageAttributes: {
    panda: {
      Type: "String",
      Value: "pushWrenchSnapshotToSTS"
    }
  }
```

Fetch all the assets in asset panda belonging to "owner", and "category", and push these assets to "site" database.
e.g. owner = STW, category = Manual Wrench, site = https:\\\sts-test.azurewebsites.net

```json
{
  "objects": 3,
  "assets": [
    {
      "id": "59c132d969702d82d8f60b00",
      "partId": "5b44ea965cdb80310f275dce",
      "modelName": "Tochnichi CEM3-G",
      "partNumber": "CEM200N3X19D-G-BTD",
      "serialNumber": "705374G",
      "bluetoothName": "ESD200v1.1.7-3105D2",
      "calibration": {
        "expiredAt": "2017-06-22T00:00:00.000Z",
        "calibratedAt": "2016-06-22T00:00:00.000Z",
        "certificate_url": ""
      },
      "firmwareRevision": null,
      "inspection": null
    },
    {
      "id": "59c132d969702d82d8f70b00",
      "partId": "5b44ea965cdb80310f275dce",
      "modelName": "Tochnichi CEM3-G",
      "partNumber": "CEM200N3X19D-G-BTD",
      "serialNumber": "705375G",
      "bluetoothName": "000190C3513B",
      "calibration": {
        "expiredAt": "2019-02-22T00:00:00.000Z",
        "calibratedAt": "2018-02-22T00:00:00.000Z",
        "certificate_url": ""
      },
      "firmwareRevision": null,
      "inspection": null
    },
    {
      "id": "59c132da69702d82d8fd0b00",
      "partId": "5b44eaa85cdb803133275e0c",
      "modelName": "Tochnichi CEM3-G",
      "partNumber": "CEM360N3X22D-G-BTD",
      "serialNumber": "707349G",
      "bluetoothName": "ESD200v1.1.7-310C4D",
      "calibration": {
        "expiredAt": "2017-10-05T00:00:00.000Z",
        "calibratedAt": "2016-10-05T00:00:00.000Z",
        "certificate_url": ""
      },
      "firmwareRevision": null,
      "inspection": null
    }
  ],
  "snapshots": 3
}
```

# Credentials

User credentials and tokens are stored in AWS System Manager (SSM) under "Parameter Store". You may click on the
key then click "Show" under "Value" to find the key's value. To set these values locally use :

```
awsudo -u sts-dev aws --region <appropriate region> ssm put-parameter --name <key name> --value <key value> --description "<appropriate key description>" --type SecureString --key-id "alias/sts-assets"
```

To use the asset-panda-gateway API directly you must set the following two keys :

```
AP_TOKEN

CONTROL_CENTER_ASSET_PANDA_GATEWAY_API_KEY
```

In addition, an API Gateway Key is provided when the repo is deployed. This key is displayed on the console by the
serverless framework as follows:

```
Service Information
service: sts-asset-panda-gateway
stage: dev
region: us-east-2
stack: sts-asset-panda-gateway-dev
api keys:
  stsAssetPanda: "actualAPIKey"
endpoints:
  GET - https://apiGatewayId.execute-api.us-east-2.amazonaws.com/dev/owner/{owner}/wrenches/assets/snapshots
  GET - https://apiGatewayId.execute-api.us-east-2.amazonaws.com/dev/part/{partid}
  GET - https://apiGatewayId.execute-api.us-east-2.amazonaws.com/dev/part/category/{category}
  GET - https://apiGatewayId.execute-api.us-east-2.amazonaws.com/dev/part/parts
  GET - https://apiGatewayId.execute-api.us-east-2.amazonaws.com/dev/tool/calibration/{id}
  GET - https://apiGatewayId.execute-api.us-east-2.amazonaws.com/dev/tool/{id}
  GET - https://apiGatewayId.execute-api.us-east-2.amazonaws.com/dev/owner/{owner}/category/{category}/asset
  POST - https://apiGatewayId.execute-api.us-east-2.amazonaws.com/dev/owner/{owner}/category/{category}/asset
  PUT - https://apiGatewayId.execute-api.us-east-2.amazonaws.com/dev/owner/{owner}/category/{category}/asset/{id}
  DELETE - https://apiGatewayId.execute-api.us-east-2.amazonaws.com/dev/owner/{owner}/category/{category}/asset/{id}
functions:
  PushWrenchSnapshotToSTS: sts-asset-panda-gateway-dev-PushWrenchSnapshotToSTS
  GetManualWrenchSnapshots: sts-asset-panda-gateway-dev-GetManualWrenchSnapshots
  GetPartSnapshot: sts-asset-panda-gateway-dev-GetPartSnapshot
  GetPartsByCategory: sts-asset-panda-gateway-dev-GetPartsByCategory
  GetAllPartSnapshots: sts-asset-panda-gateway-dev-GetAllPartSnapshots
  GetToolCalibrationDoc: sts-asset-panda-gateway-dev-GetToolCalibrationDoc
  GetPandaAssetsById: sts-asset-panda-gateway-dev-GetPandaAssetsById
  GetPandaAssets: sts-asset-panda-gateway-dev-GetPandaAssets
  AddNewAssetSnapshot: sts-asset-panda-gateway-dev-AddNewAssetSnapshot
  EditAssetSnapshot: sts-asset-panda-gateway-dev-EditAssetSnapshot
  DeleteAssetSnapshot: sts-asset-panda-gateway-dev-DeleteAssetSnapshot
layers:
  None

```

This information can also be obtained by running the following command:
```
yarn sls info -s
```

To interact with the asset-panda-gateway API directly, use the curl tool with the following template :

```
curl --<GET, PUT, POST or DELETE as appropriate for function> <end point for desired function> --header "x-api-key:<api key>"
```

# Development Environment

## Chocolatey (Windows only)

The Chocolatey package manager ensures consistent development environment
construction in Windows. Chocolatey works with Powershell. Powershell must always be
started with administrator privilege. To install Chocolatey:

1.  Press the windows button and type "powershell".
2.  Right-click on "Windows PowerShell" and select "Run as Administrator".
3.  Check the execution policy by running `Get-ExecutionPolicy`.
4.  Set the policy by running `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned`.
5.  Install Chocolatey:
    `iwr https://chocolatey.org/install.ps1 -UseBasicParsing | iex`.

## Node

This project is based on Node 8.10. We use the same version of Node as AWS Lambda, to minimize any friction due to
differences between Node releases. We are not using babel to transform the source code deployed to Lambda.

In Windows, install Node using Chocolatey from an administrator-privileged PowerShell terminal.

```ps
choco install nodejs --version 8.10.0 -y
```

In unix development environments, use the `n` node version manager. To install n:

```sh
curl -L https://git.io/n-install | bash
n 8.10.0
```

If you use zsh, add ~/n/bin to the PATH exported in zshrc.

## Yarn

The development environment requires the yarn package manager. This package manager provides a stable lock file
format when doing development work on multiple operating systems. Yarn ensure deterministic node package installation on
each platform. It is also faster than npm.

To install yarn in Windows, install yarn using choco: `choco install yarn -y` from an administrator-privileged
PowerShell terminal.

On OSX, install yarn using the homebrew package `brew install yarn --without-node`. The `--without-node` argument
allows your system to support multiple versions of node, managed by `n`.

After checking out this repo and installing yarn, run `yarn` from the command-line to install the node package
dependencies.

## Serverless Framework

The development environment requires your developer AWS credentials to be registered with the serverless framework.

```bash
yarn sls config credentials --provider aws --key <KEY> --secret <SECRET>
```

## Local execution

The following command line runs the `showVersion` function locally.

```bash
yarn sls invoke local --function showVersion
```

### Smart Torque System Production Account

The _STS Production Account_ implements the back-end for the Smart Torque System. The separate (project-specific) account prevents accidental interference from other projects, while avoiding the need to define complicated access policies for every Shell TechWorks user. The separate account can be transferred to another organization without interrupting service to users.
### Administrator Role

Developers assume the _Administrator Role_ in order to directly affect the STS Account. Configure your command-line tools like this:

```bash
aws configure --profile sts-bastion # ...copy in your AWS Key ID and Secret...
aws configure set source_profile sts-bastion --profile sts-dev
aws configure set role_arn arn:aws:iam::327603807664:role/AdministratorRole --profile sts-dev
aws configure set mfa_serial arn:aws:iam::767196065322:mfa/USERNAME --profile sts-dev
aws configure set source_profile sts-bastion --profile sts-prod
aws configure set role_arn arn:aws:iam::621640011251:role/AdministratorRole --profile sts-prod
aws configure set mfa_serial arn:aws:iam::767196065322:mfa/USERNAME --profile sts-prod
```

## Awsudo

The following command line installs awsudo.

```bash
awsudo -u sts-dev yarn sls
```

# Production Environment

The gateway is implemented using the [Serverless Framework](https://serverless.com/). The framework manages a CloudFront
stack for each deployed _stage_. The default stage is _dev_, and is meant as a sandbox for development. The production
stage is named _prod_. It is accessed from the command-line tools using the `--stage prod` argument.

## Deployment

To deploy the default (dev) stage, run the following command.

```bash
awsudo -u sts-dev yarn sls deploy --stage dev
```

For development and testing, it is possible to deploy a single function. This is quicker than deploying the full stack,
but results in an inconsistent production configuration.

```bash
yarn sls deploy function -f yourFunction
```

## Logs

Logs are collected by AWS CloudWatch. The Serverless Framework provides a convenient way to access the logs from a given
function. For example, to tail the logs of the `pollCore` function:

```bash
yarn sls logs --tail --function yourFunction
```

Logs are emitted by handlers using [lambda-log](https://github.com/KyleRoss/node-lambda-log).

Each handler module logs the package version when it is loaded. This should ensure that the package version is recorded near the beginning of each log stream.

## Removal

To remove all AWS resources for this project, run:

```bash
yarn sls remove
```

The lambda-log package outputs JSON logs for easier processing with CloudWatch Insights. It also improves the formatting of logs in dev builds.

Logging the version number can help with debugging during blue/green deployments. Outputting the version number also ensures that the Lambda source code changes with each deployment. Changing the source code is helpful to avoid the CD failure "A version for this Lambda function exists". The error occurs when the stack's CloudFormation template changes, but the source code to the lambda functions does not.

### Rationale

CloudWatch integration is unavoidable when using AWS Lambda. Chaining log streams through CloudWatch is preferable to
direct integration with a log collector because it simplifies the serverless function and probably reduces the required
IO resources per log message.

# Test

Jest is used to run tests. We expect complete unit test coverage, and Jest is configured to fail if coverage is less
than 100%. The tests should be written in a literate style, so that they help to document the intended behavior of the
software. High test coverage gives confidence that the software works as intended.

ESLint is used to enforce the airbnb code style guidelines. The burden of the style guidelines are eased through
automated correction. The Prettier package helps the IDE automatically improve the formatting of the source code. The
husky git hook automatically corrects some other ESLint issues when adding.

The project is required to be configured with a Continuous Integration server. The Continuous Integration server runs
the tests and lint using the command `yarn test`. When AppVeyor is used for CI, the jest tests are reported to the
[test result collector](https://ci.appveyor.com/project/shelltechworks/serverless/build/tests).

# Release

Our serverless framework projects have a rudimentary deployment pipeline. A deployment is started when a version tag
(like "v1.2.3") is pushed to GitHub. The CI process is triggered by the tag. If the build passes, then CI creates a
release on GitHub. Finally CI deploys to the _prod_ stage. At this time, we do not have steps for end-to-end testing or
SQA review.

Normal releases are made from the master branch. To bump the version number and start the process of publishing a
release, run `yarn publish:patch`. This will set the package.json version to the next patch number and push a
tagged commit, triggering the deployment process.

It is possible to make a _hotfix_ from a branch other than master, but do not to make a habit of it.

