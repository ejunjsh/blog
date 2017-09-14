---
title: 利用树莓派实现一个能播放天气的闹钟
date: 2017-07-18 01:12:25
tags: [raspberrypi,python]
categories: raspberrypi
---
# 前提
你要有个pi😄
<!-- more -->
# 获取天气接口
这里我是用图灵机器人来获取天气的接口，你可以自己上去注册一个，下面代码URL的Key是我注册的机器人给的。
````python
def getWeatherText():
    try:
        response = requests.get(
            "http://www.tuling123.com/openapi/api?key=652ae4a714794fe6b01faa990d7a981f&info=%s" % "广州今日天气")
        json = response.json()
        if json["code"] == 100000:
            return json["text"]
        else:
            return ""
    except:
        return ""
````
# 播放文字
利用百度的接口可以转换文本为语音。默认只有女声
````python
def text2voice(text):
    url = 'http://tts.baidu.com/text2audio?idx=1&tex={0}&cuid=baidu_speech_' \
          'demo&cod=2&lan=zh&ctp=1&pdt=1&spd=4&per=4&vol=5&pit=5'.format(text)
    # 用mplayer播放语音
    os.system('mplayer "%s"' % url)
````

# 安装播放媒体软件
上面代码你看到的`mplayer`,就是用来播放语音的，传个url作为参数
````bash
sudo apt-get install mplayer
usage: mplayer [url]
````

# 播放音乐
有了上面这个神器，你可以给播报语音前后加一首音乐😄
````python
def playMusic(path):
    os.system('mplayer %s' % path)
````

# 总结
利用上面的东东，可以组合些好玩的东西了，至于闹钟的唤醒，可以cron job 做，也可以代码里面实现，enjoy...😄
全部代码地址 https://github.com/ejunjsh/raspberrypi-code/blob/master/clock/weather.py