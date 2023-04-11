![alt text](https://fourtf.com/img/chatterino-icon-64.png)
Chatterino Red - A fork of Chatterino 2
============

Chatterino 2 is a chat client for [Twitch.tv](https://twitch.tv).
The Chatterino 2 wiki can be found [here](https://wiki.chatterino.com). Visit the original repository [here](https://github.com/Chatterino/chatterino2).
 Chatterino Red allows you to use a other process as a proxy for the chat.

## Download

Current releases are available at the [releases tab](https://github.com/dodaucy/chatterino-red/releases).

## Building

To get source code with required submodules run:

```
git clone --recurse-submodules https://github.com/dodaucy/chatterino-red.git
```

or

```
git clone https://github.com/dodaucy/chatterino-red.git
cd chatterino-red
git submodule update --init --recursive
```

[Building on Windows](../master/BUILDING_ON_WINDOWS.md)

[Building on Windows with vcpkg](../master/BUILDING_ON_WINDOWS_WITH_VCPKG.md)

[Building on Linux](../master/BUILDING_ON_LINUX.md)

[Building on Mac](../master/BUILDING_ON_MAC.md)

[Building on FreeBSD](../master/BUILDING_ON_FREEBSD.md)

## Configuration

Chatterino Red uses a JSON file for configuration that is located in the Chatterino Settings folder. The Chatterino Settings folder is located at:

On **Windows**: `%APPDATA%/Chatterino2`

On **Linux**: `$HOME/.local/share/chatterino`

On **Mac**: `$HOME/Library/Application Support/chatterino`

Open this folder and create a file called `red_settings.json`. The file should contain a `proxy_url` key with the url of the proxy as value. For example:

```json
{
    "proxy_url": "http://127.0.0.1:8000"
}
```

## Proxy

The proxy should be a HTTP server that simply waits for a POST request on `/`. The request contains JSON data with the following keys:

- `channel`: The channel name

- `message`: The actual message

You can then do whatever you want and return an action as an response. The response should be a JSON object with one of the following actions as a value for the `action` key:

- `send`: You need to return a `channel` and `message` key with the channel and message you want to send. You can easily use the original channel and message when you want to send the message normal.

- `ignore`: You can ignore the message by not returning this.

- `system_message`: You can send a system message by returning a `message` key with the message you want to send. This message will be sent in the Chatterino Client.

### Example

You need to install `pip install fastapi[all] pydantic` to run this example with `uvicorn SCRIPT_NAME:app`.

```python
from fastapi import FastAPI
from pydantic import BaseModel


class Message(BaseModel):
    channel: str
    message: str


app = FastAPI()


@app.post("/")
def index(message: Message):
    print(f"{message.channel}: {message.message}")
    if message.message == "Please ignore me":
        # Ignore the message
        return {
            "action": "ignore"
        }
    elif message.message.startswith("!system "):
        # Send a system message
        return {
            "action": "system_message",
            "message": message.message.split(" ", 1)[1]
        }
    else:
        # Send the message to the channel
        return {
            "action": "send",
            "message": message.message,
            "channel": message.channel
        }
```

<video controls>
  <source src="example.mp4" type="video/mp4">
</video>
