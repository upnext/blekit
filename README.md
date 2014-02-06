BLEKit is a framework for efficiently managing large-scale deployments of beacons (Bluetooth Low Energy devices implementing Apple’s iBeacon protocol). With a simple yet expressive JSON-based syntax, you can easily specify typical beacon actions, such as sending welcome messages, executing remote-server actions and performing automatic checkins. Additionally, BLEKit allows you to describe more complex expressions such as “IF user enters the immediate range of a beacon AND stays there for 1 minute THEN send him a message”. All this can be achieved through JSON configuration without the need to write a single line of code.

BLEKit has been designed to be easily expandable with custom actions. The library offers a collection of common and reusable items, but is also well-suited to serve as a base layer for building custom service interactions.

BLEKit is available for iOS and Android. API documentation and specific integration instructions can be found in these sub-projects:

* [BLEKit for iOS](https://github.com/upnext/blekit-ios)
* [BLEKit for Android](https://github.com/upnext/blekit-android)

The following documentation describes the common parts: basic models and semantic items used in the library + the JSON format.

## Basic concepts

### Beacon

A beacon is a simple Bluetooth Low Energy device that broadcasts data according to Apple’s iBeacon protocol. The BLEKit library is specifically designed to work with the iBeacon protocol and the location APIs available in iOS7.

### Zone

A zone is a collection of beacons typically representing one logical place such as a shop or a venue area. Zones have names, physical locations (coordinates) and other useful metadata.

### Action 

As the user enters the zone and interacts with beacons, actions are executed. Actions include UI-visible tasks such as sending a message or presenting content… but also feature background tasks such as performing http requests and interacting with remote APIs.

### Conditions

Conditions describe requirements which need to be satisfied in order to perform actions. Each action is associated with one or more conditions. A typical condition is “enter” (user entered), or “stays” (user stayed X time).

## JSON format

A schema for the JSON cofiguration file.
	
	{
    "type": "object",
    "properties": {
      "id": {
        "type": "string"
      },
      "name": {
        "type": "string"
      },
      "ttl": {
        "type": "integer"
      },
      "radius": {
        "type": "number"
      },
      "beacons": {
        "type": "array",
        "items": [
          {
            "type": "object",
            "properties": {
              "id": {
                "type": "string"
              },
              "description": {
                "type": "string"
              },
              "name": {
                "type": "string"
              },
              "location": {
                "type": "object",
                "properties": {
                  "latitude": {
                    "type": "number"
                  },
                  "longitude": {
                    "type": "number"
                  }
                },
                "additionalProperties": false
              },
              "triggers": {
                "type": "array",
                "items": [
                  {
                    "type": "object",
                    "properties": {
                      "id": {
                        "type": "integer"
                      },
                      "name": {
                        "type": "string"
                      },
                      "comment": {
                        "type": "string"
                      },
                      "action": {
                        "type": "object",
                        "properties": {
                          "id": {
                            "type": "integer"
                          },
                          "type": {
                            "type": "string"
                          },
                          "parameters": {
                            "type": "object",
                            "optional": true,
                            "additionalProperties": false
                          }
                        },
                        "additionalProperties": false
                      },
                      "conditions": {
                        "type": "array",
                        "items": [
                          {
                            "type": "object",
                            "properties": {
                              "id": {
                                "type": "integer"
                              },
                              "type": {
                                "type": "string"
                              },
                              "parameters": {
                                "type": "object",
                                "optional": true,
                                "additionalProperties": false
                              },
                              "expression": {
                                "type": "string",
                                "optional": true
                              }
                            },
                            "additionalProperties": false
                          }
                        ],
                        "additionalProperties": false
                      }
                    },
                    "additionalProperties": false
                  }
                ],
                "additionalProperties": false
              }
            },
            "additionalProperties": false
          }
        ],
        "additionalProperties": false
      },
      "location": {
        "type": "object",
        "properties": {
          "latitude": {
            "type": "number"
          },
          "longitude": {
            "type": "number"
          }
        },
        "additionalProperties": false
      }
    },
    "additionalProperties": false
	}
	
### Zone

This is the root of the configuration file.
It describes the area in which beacons are placed.

| Field name | Value type | Description |
| ------------ | ------------- | ------------ |
| id | string  | identifier of the zone |
| name | string  | name of the zone |
| ttl | integer  | time to live; after this amount of seconds configuration should be re-fetched |
| radius | number  | radius of the zone in metres |
| location | Location  | location of the zone |
| beacons | array of Beacon  | array of beacons |

### Location

Simple location containing coordinates.

| Field name | Value type | Description |
| ------------ | ------------- | ------------ |
| latitude| number  | latitude of location |
| longitude| number  | longitude of location |

### Beacon

Represents a physical Beacon. 

| Field name | Value type | Description |
| ------------ | ------------- | ------------ |
| id | string  | beacon identifier; it is a composition of proximityUUID+major+minor, eg. D57092AC-DFAA-446C-8EF3-C81AA22815B5+5+5000  |
| description | string  | description for the beacon |
| name | string  | name of the beacon |
| location | Location  | location of the beacon |
| triggers | array of Trigger  | array of triggers |

The most important property is the *id* - a composition of beacons proximityUUID, major and minor separated by a '+' sign.
It is mandatory that UUID is a 36-chars string. Major and minor can be empty, and then all beacons matching the given UUID will be discovered.

A beacon can have several triggers.

### Trigger

A trigger is a set of conditions that have to be fulfilled in order for an action to be performed.

| Field name | Value type | Description |
| ------------ | ------------- | ------------ |
| id | integer | trigger identifier |
| name | string | trigger name |
| comment | string | trigger comment |
| action | Action | action to perform |
| conditions | array of Condition | conditions that have to be fulfilled |

### Condition

Describes a condition that has to be fulfilled.

The *type* parameter describes the condition evaluator - it is matched against existing implemetations available on the client-side (either Android or iOS). There are several implementations available out-of-the box described [here](http://link.to.conditions).

Each implementation of conditionrequires (or not) specific parameters.

If the *expression* parameter is not *null* then it has precedence in evaluation over regular condition evaluation.

Expression is a javascript logic formula that might look like:

	zone.name = "Upnext HQ" && trigger.type = "enter"

| Field name | Value type | Description |
| ------------ | ------------- | ------------ |
| id | integer | condition identifier |
| type | string | condition type |
| parameters | object | optional object with parameters |
| expression | string | optional expression to evaluate |


### Action

| Field name | Value type | Description |
| ------------ | ------------- | ------------ |
| id | integer | action identifier |
| type | string | action type |
| parameters | object | optional object with parameters |


