# Interface to Websocket API

The Repository containing the code of the Websocket API can be found on [Git](https://github.com/VirtualProgrammingLab/viplab-websocket-api).

The Websocket API is based on JSON messages. 
Those messages are sent between the browser and the server. 
Each of the messages has a type and a content, whereby the type determines how the content of the message is processed. 

``` json title="JSON Message"
{
    "$schema": "http://json-schema.org/draft-07/schema",
    "$id": "message-schema.json",
    "title": "Message",
    "description": "Message format of the ViPLab Websocket API",
    "type": "object",
    "properties": {
        "type": {
            "type": "string",
            "description": "The type of the message"
        },
        "content": {
            "type": "object",
            "description": "The content of the message"
        }
    },
    "required": [
        "type",
        "content"
    ]
}
```

## Message Types

### Messages from Browser to Server

#### authenticate

After establishing a WebSocket connection, authentication takes place. 
Therefore the first message sent ist the authenticate message. 
This message contains authentication tokens in terms of JWTs, that are provided to the browser by an external service. 

``` json title="authenticate Message Example"
{
    "type": "authenticate",
    "content": {
        "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
    }
}
```

#### create-computation

Message to create a new [Computation](../viplab3.0/computation.md) from a [Computation Template](../viplab3.0/computation_template.md) and a [Computation Task](../viplab3.0/computation_task.md). 
This Computation is sent back to the browser using a [computation message](#computation)

#### subscribe

Using this message, one can subscribe to different events. 
For subscribing to an event, the respective event needs to be passed in the message content as string in the topic property. 
The string is separated by a colon. 
The message is idempotent, thus you can only subscribe to an event once per WebSocket session. 
No response is sent back from the server to confirm the subscription.

| Event | Description |
| ----- | ----------- |
| `computation:{id}` | All events about one computation |

``` json title="subscribe Message Example"
{
    "type": "subscribe",
    "content": {
        "topic": "computation:b30fca08-7267-4aa9-adc7-8c89f5efd928"
    }
}
```

### Messages from Server to Browser

#### computation

This message is used to send a Computation back to the browser.

``` json title="computation Message Example"
{
    "type": "computation",
    "content": {
        "id": "20f32dc9-e43f-4a65-af8c-42412ed9d977",
        "created": "2019-08-19T13:31:30+0200",
        "expires": "2019-08-19T16:31:30+0200",
        "status": "created"
    }
}
```