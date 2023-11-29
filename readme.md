# Readme
codes for Calculator Project

- [Installation](#installation)
- [Usage](#usage)
- [소스코드](#소스코드)
-  [Uninstallation](#uninstallation)
설치방법과 사용방법 및 프로그램 소개를 달아서 프로그램의 개요와 사용법을 간략하게 알도록 썼습니다. 또한 주요한 소스코드들을 적어 어떤 코드를 썼는지 알 수 있게 써 놓았고 마지막으로 제거 방법을 써서 프로그램 종료와 삭제 방법을 기제하였습니다.   
  
# Installation

Install telegram-send system-wide with pip:
``` shell
sudo pip3 install telegram-send
```

Or if you want to install it for a single user without root permissions:
``` shell
pip3 install telegram-send
```

If installed for a single user you need to add `~/.local/bin` to their path,
refer to this [guide][] for instructions.

And finally configure it with `telegram-send --configure` if you want to send to
your account, `telegram-send --configure-group` to send to a group or with
`telegram-send --configure-channel` to send to a channel.

Use the `--config` option to use multiple configurations. For example to set up
sending to a channel in a non-default configuration: `telegram-send --config
channel.conf --configure-channel`. Then always specify the config file to use
it: `telegram-send --config channel.conf "Bismillah"`.

The `-g` option uses the global configuration at `/etc/telegram-send.conf`.
Configure it once: `sudo telegram-send -g --configure` and all users on the
system can send messages with this config: `telegram-send -g "GNU"` (provided
you've installed it system-wide.)

[guide]: https://www.rahielkasim.com/installing-programs-from-non-system-package-managers-without-sudo/
---
# Usage

To send a message:
``` shell
telegram-send "Hello, World!"
```
There is a maximum message length of 4096 characters, larger messages will be
automatically split up into smaller ones and sent separately.

To send a message using Markdown or HTML formatting:
```shell
telegram-send --format markdown "Only the *bold* use _italics_"
telegram-send --format html "<pre>fixed-width messages</pre> are <i>also</i> supported"
telegram-send --format markdown "||Do good and find good!||"
```
Note that not all Markdown syntax or all HTML tags are supported. For more
information on supported formatting, see the [formatting options][]. We use the
MarkdownV2 style for Markdown.

[formatting options]: https://core.telegram.org/bots/api#formatting-options

The `--pre` flag formats messages as fixed-width text:
``` shell
telegram-send --pre "monospace"
```

To send a message without link previews:
``` shell
telegram-send --disable-web-page-preview "https://github.com/rahiel/telegram-send"
```

To send a message from stdin:
``` shell
printf 'With\nmultiple\nlines' | telegram-send --stdin
```
With this option you can send the output of any program.

To send a file (maximum file size of 50 MB) with an optional caption:
``` shell
telegram-send --file quran.pdf --caption "The Noble Qur'an"
```

To send an image (maximum file size of 10 MB) with an optional caption:
``` shell
telegram-send --image moon.jpg --caption "The Moon at Night"
```

To send a GIF or a soundless MP4 video (encoded as H.264/MPEG-4 AVC with a maximum file size of 50 MB) with an optional caption:
``` shell
telegram-send --animation kitty.gif --caption "🐱"
```

To send an MP4 video (maximum file size of 50 MB) with an optional caption:
``` shell
telegram-send --video birds.mp4 --caption "Singing Birds"
```

To send an audio file with an optional caption:
``` shell
telegram-send --audio "Pachelbel's Canon.mp3" --caption "Johann Pachelbel - Canon in D"
```

To send a location via latitude and longitude:
``` shell
telegram-send --location 35.5398033 -79.7488965
```

All captions can be optionally formatted with Markdown or html:
``` shell
telegram-send --image moon.jpg --caption "The __Moon__ at *Night*" --format markdown
```

# 주요 소스코드  
```python  
#See the result of this script in the channel: https://telegram.me/astropod
import argparse
import json
import os
from subprocess import call

import requests

from secret import key


def main():
    parser = argparse.ArgumentParser(description="Send the daily Astronomy Picture of the Day.")
    parser.add_argument("--config", help="configuration file for telegram-send", type=str)
    args = parser.parse_args()
    conf_command = ["--config", args.config] if args.config else []

    api = "https://api.nasa.gov/planetary/apod"
    payload = {"api_key": key}  # date: YYYY-MM-DD
    r = requests.get(api, params=payload)

    data = json.loads(r.text)
    url = data["url"]
    explanation = data["explanation"]
    title = data["title"]

    year, month, day = data["date"].split('-')
    link = "http://apod.nasa.gov/apod/ap" + year[2:] + month + day + ".html"

    if data["media_type"] == "image":
        if "hdurl" in data:
            hdurl = data["hdurl"]
            hd_headers = requests.head(hdurl).headers
            if int(hd_headers["Content-Length"]) < 2E6:  # only download HD images under 2 MB
                url = hdurl
        image = requests.get(url).content

        filename = "astro"
        with open(filename, "wb") as f:
            f.write(image)
        call(["telegram-send", "--image", filename, "--caption", title + " - " + link] + conf_command)
        os.remove(filename)
    elif data["media_type"] == "video":
        message = url
        call(["telegram-send", message] + conf_command)


if __name__ == "__main__":
    main()

```
---    
# Uninstallation

``` shell
sudo telegram-send --clean
sudo pip3 uninstall telegram-send
```
