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

Anova uses a Firebase application to authenticate its users with a password.

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

You must connect to `wss://app.oven.anovaculinary.io`.

## Authentication Command

```json
{
  "command": "AUTH_TOKEN",
  "payload": "[ID_TOKEN]",
}
```

> The service will respond with the following message

```json
{"response":"AUTH_TOKEN_RESPONSE"}
```

In order to authenticate, you must send a message containing the `idToken` you received in the
earlier authorization call as the payload.

## Send Command

```json
{
  "command": "SEND_OVEN_COMMAND",
  "payload": {
    "deviceId": "<DEVICE_ID>",
    "command": <COMMAND>,
    "requestId": "<REQUEST_UUID>"
  }
}
```

> The service will acknowledge the message with the following response

```json
{
  "response": "OVEN_COMMAND_RESPONSE",
  "requestId": "<REQUEST_UUID>",
  "deviceId": "<DEVICE_ID>",
  "success": true,
  "data": [{},null,null]
}
```

### Parameters

Parameter | Description
--------- | -----------
REQUEST_UUID | A UUID to differentiate this request from others.
COMMAND | One of the Commands below
DEVICE_ID | A unique identifer for the specific oven. In order to find the device ids that are available to you, you can wait for an <a href="#oven-state">Oven State message</a> to be received.

# Send Command Payloads

## Start Cook

```json
{
	"id": "<COMMAND_UUID>",
	"type": "startCook",
	"payload": {
		"cookId": "<DEVICE_ID>",
		"stages": [
			{
				"stepType": "stage",
				"id": "<STAGE_UUID>",
				"title": "",
				"description": "",
				"type": "preheat",
				"userActionRequired": false,
				"temperatureBulbs": {
					"wet": {
						"setpoint": {
							"fahrenheit": 131,
							"celsius": 55
						}
					},
					"mode": "wet"
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
	}
}
```

This must be sent within a <a href="#send-command">Send command</a>.

Parameter | Description
--------- | -----------
COMMAND_UUID | A UUID to differentiate this command from others.
DEVICE_ID | A unique identifer for the specific oven. In order to find the device ids that are available to you, you can wait for an <a href="#oven-state">Oven State message</a> to be received.
STAGE_UUID | A unique identifer to differentiate this cooking stage from others.

## Stop Cook

```json
{
	"id": "<UUID>",
	"type": "stopCook"
}
```

This must be sent within a <a href="#send-command">Send command</a>.

Parameter | Description
--------- | -----------
UUID | A UUID to differentiate this command from others.

# Websocket Messages

## Oven State

The oven will broadcast periodically with it's state.

```json
{
  "response": "OVEN_STATE",
  "ovenId": "<DEVICE_ID>",
  "data": {
    "version": 1,
    "updatedTimestamp": "2023-02-07T05:37:52Z",
    "systemInfo": {
      "online": true,
      "hardwareVersion": "120V Universal",
      "powerMains": 120,
      "powerHertz": 60,
      "firmwareVersion": "1.4.24",
      "uiHardwareVersion": "UI_RENASAS",
      "uiFirmwareVersion": "1.0.22",
      "firmwareUpdatedTimestamp": "2022-12-29T20:21:49Z",
      "lastConnectedTimestamp": "2023-02-07T02:55:23Z",
      "lastDisconnectedTimestamp": "2023-02-07T02:55:17Z",
      "triacsFailed": false
    },
    "state": {
      "mode": "cook",
      "temperatureUnit": "F",
      "processedCommandIds": [
        "d98bcdfd-0609-4cce-8123-a79f744505b2",
        "71102b16-27f5-4962b186-02a6d8580aac",
        "51b5bf15-1fb4-4352-9408-24f282c79e79",
        "d124f649-0112-4d36-8a33-90589c1f7e80",
        "937e8575-8cb8-4372-aa39-086f77c6a3d4",
        "f407ea2c-c31a-4b1a-8847-3b92313add97",
        "f563a89f-44f6-4caa-bced-68a625569e7a",
        "7799d35a-bea4-4e5f-bb8b-78921a81dd73",
        "d17bc3ab-162a-47d4afb9-789a92e6975f",
        "3943226a-31cf-4e94-8374-9e6a868cf255"
      ]
    },
    "nodes": {
      "temperatureBulbs": {
        "mode": "wet",
        "wet": {
          "current": {
            "celsius": 25.45,
            "fahrenheit": 77.81
          },
          "setpoint": {
            "celsius": 55,
            "fahrenheit": 131
          },
          "dosed": true,
          "doseFailed": false
        },
        "dry": {
          "current": {
            "celsius": 25.45,
            "fahrenheit": 77.81
          }
        },
        "dryTop": {
          "current": {
            "celsius": 25.45,
            "fahrenheit": 77.81
          },
          "overheated": false
        },
        "dryBottom": {
          "current": {
            "celsius": 24.98,
            "fahrenheit": 76.96
          },
          "overheated": false
        }
      },
      "timer": {
        "mode": "idle",
        "initial": 0,
        "current": 0
      },
      "temperatureProbe": {
        "connected": false
      },
      "steamGenerators": {
        "mode": "relative-humidity",
        "relativeHumidity": {
          "current": 100,
          "setpoint": 100
        },
        "evaporator": {
          "failed": false,
          "overheated": false,
          "celsius": 38.72,
          "watts": 0
        },
        "boiler": {
          "descaleRequired": false,
          "failed": false,
          "overheated": false,
          "celsius": 38.72,
          "watts": 0,
          "dosed": false
        }
      },
      "heatingElements": {
        "top": {
          "on": false,
          "failed": false,
          "watts": 0
        },
        "bottom": {
          "on": false,
          "failed": false,
          "watts": 0
        },
        "rear": {
          "on": true,
          "failed": false,
          "watts": 0
        }
      },
      "fan": {
        "speed": 100,
        "failed": false
      },
      "vent": {
        "open": true
      },
      "waterTank": {
        "empty": false
      },
      "door": {
        "closed": true
      },
      "lamp": {
        "on": true,
        "failed": false,
        "preference": "on"
      },
      "userInterfaceCircuit": {
        "communicationFailed": false
      }
    },
    "cook": {
      "cookId": "<COMMAND_UUID>",
      "stages": [
        {
          "title": 0,
          "id": "<STAGE_UUID>",
          "type": "preheat",
          "userActionRequired": false,
          "temperatureBulbs": {
            "mode": "wet",
            "wet": {
              "setpoint": {
                "celsius": 55,
                "fahrenheit": 131
              }
            }
          },
          "steamGenerators": {
            "mode": "relative-humidity",
            "relativeHumidity": {
              "setpoint": 100
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
          }
        },
        {
          "title": 0,
          "id": "<STAGE_UUID>",
          "type": "cook",
          "userActionRequired": false,
          "temperatureBulbs": {
            "mode": "wet",
            "wet": {
              "setpoint": {
                "celsius": 55,
                "fahrenheit": 131
              }
            }
          },
          "steamGenerators": {
            "mode": "relative-humidity",
            "relativeHumidity": {
              "setpoint": 100
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
          }
        }
      ],
      "activeStageId": "<STAGE_UUID>",
      "activeStageIndex": 0,
      "stageTransitionPendingUserAction": false,
      "activeStageSeconecondsElapsed": 1
    }
  }
}
```

Parameter | Description
--------- | -----------
DEVICE_ID | A unique identifer for the specific oven.
COMMAND_UUID | A unique identifer to differentiate this command from others.
STAGE_UUID | A unique identifer to differentiate this cooking stage from others.
