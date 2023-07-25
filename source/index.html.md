---
title: API Reference

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers

toc_footers:

includes:

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Kittn API
---

# Introduction

This is the private API used by the Anova Precision Oven mobile applications.

## Changelog
- 7/25/2023: Updated to Anova Oven API V2 (oven firmware 2+)

# Authentication

```shell
curl 'https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=AIzaSyCGJwHXUhkNBdPkH3OAkjc9-3xMMjvanfU' \
-H 'Content-Type: application/json' \
--data-binary '{"email":"user@example.com","password":"PASSWORD","returnSecureToken":true}'
```

```json
{
   "idToken": "",
   "refreshToken": "",
   "expiresIn": "",
}
```

> Be sure to store the `idToken` as you'll need it to authenticate to the
> websocket. Also note that it's a JWT token, so you can extract additional
> information about the token if you're interested.

Anova uses a Firebase application to [authenticate] its users with a password.

`POST https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
key | true | The Firebase application key

### Body Parameters

Parameter | Required | Description
--------- | ------- | -----------
email | true | The email of the Anova mobile application user
password | true | The password for the Anova mobile application user
returnSecureToken | true | Informs the API to return a JWT token


# Websocket Commands

You must connect to `wss://devices.anovaculinary.io/?token=idToken&supportedAccessories=APO&platform=android`. Replace `idToken` with the JWT Token returned by the authentication step. Specify `ANOVA_V2` as the protocol when opening the connection.

Example JavaScript code to open the WebSocket:

```js
const ws = new WebSocket(`wss://devices.anovaculinary.io/?token=${data.idToken}&supportedAccessories=APO&platform=android`, 'ANOVA_V2');
```

## Send Command

This is the overall structure of a command message to the API:

```json
{
  "command": "<COMMAND>",
  "payload": {
    "id": "<DEVICE_ID>",
    "type": "<COMMAND>",
    "payload": "<START_COOK_PAYLOAD>" # only used for start cook command, undefined otherwise
  },
  "requestId": "<REQUEST_UUID>"
}
```

> The service will acknowledge the message with the following response

```json
{
  "command": "RESPONSE",
  "requestId": "<REQUEST_UUID>",
  "payload": {
    "status": "ok" 
  }
}
```

### Parameters

Parameter | Description
--------- | -----------
REQUEST_UUID | A UUID to differentiate this request from others.
COMMAND | One of the Commands below
DEVICE_ID | A unique identifer for the specific oven. In order to find the device ids that are available to you, you can wait for an <a href="#oven-list">Oven list message</a> or an <a href="#oven-state">Oven State message</a> to be received.

# Send Command Payloads

## Start Cook
This example payload *includes* the <a href="#send-command">Send Command</a> container.

```json
{
  "command": "CMD_APO_START",
  "payload": {
    "payload": {
      "cookId": "COOK_UUID",
      "stages": [
        {
          "stepType": "stage",
          "id": "STAGE_UUID",
          "title": "",
          "description": "",
          "type": "cook",
          "userActionRequired": false,
          "temperatureBulbs": {
            "mode": "wet",
            "wet": {
              "setpoint": {
                "celsius": 54.44,
                "fahrenheit": 130
              }
            }
          },
          "heatingElements": {
            "top": {
              "on": false
            },
            "bottom": {
              "on": false
            },
            "rear": {
              "on": true
            }
          },
          "fan": {
            "speed": 100
          },
          "vent": {
            "open": false
          },
          "rackPosition": 3,
          "steamGenerators": {
            "relativeHumidity": {
              "setpoint": 100
            },
            "mode": "relative-humidity"
          }
        }
      ]
    },
    "type": "CMD_APO_START",
    "id": "<DEVICE_ID>"
  },
  "requestId": "<REQUEST_UUID>"
}
```


Parameter | Description
--------- | -----------
COMMAND_UUID | A UUID to differentiate this command from others.
DEVICE_ID | A unique identifer for the specific oven. In order to find the device ids that are available to you, you can wait for an <a href="#oven-state">Oven State message</a> to be received.
COOK_ID | A unique identifier to differentiate this cook from other cooks.
STAGE_UUID | A unique identifer to differentiate this cooking stage from others.

## Stop Cook
This example payload *includes* the <a href="#send-command">Send Command</a> container.

```json
{
  "command": "CMD_APO_STOP",
  "payload": {
    "type": "CMD_APO_STOP",
    "id": "<DEVICE_ID>"
  },
  "requestId": "<REQUEST_UUID>"
}
```

Parameter | Description
--------- | -----------
REQUEST_UUID | A UUID to differentiate this request from others.
DEVICE_ID | A unique identifer for the specific oven. In order to find the device ids that are available to you, you can wait for an <a href="#oven-state">Oven State message</a> to be received.

# Websocket Messages

## Oven List
When you open a web socket, and periodically as the list of ovens change, the service will broadcast a list of ovens that includes the name and ID of each.

```json
{
  "command": "EVENT_APO_WIFI_LIST",
  "payload": [
    {
      "cookerId": "<DEVICE_ID>",
      "name": "<OVEN_NAME>",
      "pairedAt": "2023-07-25T12:46:48.538Z",
      "type": "oven_v1"
    }
  ]
}
```

## Oven State

The oven will broadcast periodically with its state. This message appears to be sent every 30 seconds while a socket is open (regardless of activity).

```json
{
  "command": "EVENT_APO_STATE",
  "payload": {
    "cookerId": "<DEVICE_ID>",
    "type": "oven_v1",
    "state": {
      "cook": {
        "activeStageId": "<STAGE_UUID>
        "stages": [
          {
            "fan": {
              "speed": 100
            },
            "heatingElements": {
              "rear": {
                "on": true
              },
              "top": {
                "on": false
              },
              "bottom": {
                "on": false
              }
            },
            "temperatureBulbs": {
              "wet": {
                "setpoint": {
                  "fahrenheit": 130,
                  "celsius": 54.44
                }
              },
              "mode": "wet"
            },
            "steamGenerators": {
              "mode": "relative-humidity",
              "relativeHumidity": {
                "setpoint": 100
              }
            },
            "userActionRequired": false,
            "id": "<STAGE_UUID>",
            "type": "preheat",
            "vent": {
              "open": false
            },
            "title": 0
          },
          {
            "heatingElements": {
              "rear": {
                "on": true
              },
              "top": {
                "on": false
              },
              "bottom": {
                "on": false
              }
            },
            "temperatureBulbs": {
              "wet": {
                "setpoint": {
                  "celsius": 54.44,
                  "fahrenheit": 130
                }
              },
              "mode": "wet"
            },
            "id": "<STAGE_UUID>",
            "type": "cook",
            "fan": {
              "speed": 100
            },
            "steamGenerators": {
              "mode": "relative-humidity",
              "relativeHumidity": {
                "setpoint": 100
              }
            },
            "userActionRequired": false,
            "vent": {
              "open": false
            },
            "title": 0
          }
        ],
        "activeStageSecondsElapsed": 37,
        "secondsElapsed": 37,
        "cookId": "<COOK_UUID>",
        "activeStageIndex": 0,
        "stageTransitionPendingUserAction": false
      },
      "state": {
        "processedCommandIds": [
          "c3ff2a79-c4ad-4b8d-8541-cff888ee6e63",
          "1d26f0db-3b88-49da-95e0-0a02650337f6",
          "3f3aa1af-80c7-468a-b5bc-f655a13db621",
          "981ff4b5-e192-4d6d-8781-ddbad8edd69f",
          "7286d4b0-fd98-4aeb-81f4-c3aab965076d",
          "39b1c6f7-26ee-4291-b2fa-8976729dc25a",
          "038d47b2-db5c-4999-a53c-f63dceab74ff",
          "2d4d289d-6c15-4477-b1a4-3f17fbdcfbe4",
          "b0f6c33e-c8ff-4517-b8f6-36d7125cb51b",
          "b54fe16a-f200-44cb-8ab7-09330d72f0cb"
        ],
        "temperatureUnit": "F",
        "mode": "cook"
      },
      "nodes": {
        "temperatureBulbs": {
          "dryBottom": {
            "current": {
              "celsius": 28.75,
              "fahrenheit": 83.75
            },
            "overheated": false
          },
          "dryTop": {
            "current": {
              "celsius": 28.94,
              "fahrenheit": 84.1
            },
            "overheated": false
          },
          "wet": {
            "current": {
              "celsius": 28.3,
              "fahrenheit": 82.93
            },
            "dosed": false,
            "setpoint": {
              "celsius": 54.44,
              "fahrenheit": 129.99
            },
            "doseFailed": false
          },
          "dry": {
            "current": {
              "fahrenheit": 84.1,
              "celsius": 28.94
            }
          },
          "mode": "wet"
        },
        "steamGenerators": {
          "mode": "relative-humidity",
          "relativeHumidity": {
            "current": 97,
            "setpoint": 100
          },
          "boiler": {
            "watts": 0,
            "dosed": false,
            "descaleRequired": false,
            "failed": false,
            "celsius": 38.43,
            "overheated": false
          },
          "evaporator": {
            "watts": 0,
            "overheated": false,
            "failed": false,
            "celsius": 38.43
          }
        },
        "lamp": {
          "failed": false,
          "preference": "off",
          "on": false
        },
        "heatingElements": {
          "top": {
            "watts": 0,
            "on": false,
            "failed": false
          },
          "rear": {
            "watts": 0,
            "on": true,
            "failed": false
          },
          "bottom": {
            "watts": 0,
            "failed": false,
            "on": false
          }
        },
        "userInterfaceCircuit": {
          "communicationFailed": false
        },
        "timer": {
          "mode": "idle",
          "current": 0,
          "initial": 0
        },
        "fan": {
          "failed": false,
          "speed": 100
        },
        "vent": {
          "open": true
        },
        "temperatureProbe": {
          "connected": false
        },
        "door": {
          "closed": true
        },
        "waterTank": {
          "empty": false
        }
      },
      "systemInfo": {
        "powerHertz": 60,
        "lastDisconnectedTimestamp": "2023-07-25T05:57:53Z",
        "triacsFailed": false,
        "uiFirmwareVersion": "1.0.22",
        "lastConnectedTimestamp": "2023-07-25T11:58:06Z",
        "online": true,
        "firmwareVersion": "2.0.11",
        "uiHardwareVersion": "UI_RENASAS",
        "powerMains": 120,
        "hardwareVersion": "120V Universal",
        "firmwareUpdatedTimestamp": "2023-07-10T08:56:47Z"
      },
      "version": 1,
      "updatedTimestamp": "2023-07-25T12:54:07Z"
    }
  }
}
```

Parameter | Description
--------- | -----------
DEVICE_ID | A unique identifer for the specific oven.
COMMAND_UUID | A unique identifer to differentiate this command from others.
STAGE_UUID | A unique identifer to differentiate this cooking stage from others.

[authenticate]: https://firebase.google.com/docs/reference/rest/auth/#section-sign-in-email-password
