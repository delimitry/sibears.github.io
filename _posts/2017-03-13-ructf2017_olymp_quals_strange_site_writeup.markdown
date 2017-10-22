---
layout: post
title:  "RuCTF-2017 Olymp Quals, Странный сайт"
date:   2017-03-13 13:34:23 +0700
categories: Разборы
permalink: /labs/ructf2017_olymp_quals_strange_site_writeup/
author: Deflate
tags: [writeup, RuCTF-2017]
---

Описание таска:


Мы нашли интересный сайт на просторах интернета. Но есть что-то странное в его ответах. Более того, его владелец очень не любит непрошенных гостей. Твоя задача — найти флаг.
Переходим по ссылке http://deep.ructf.org/, в HTTP-ответе видим что-то необычное:

{% highlight http %}
< GET / HTTP/1.1
< Host: deep.ructf.org
< Accept-Encoding: gzip, deflate
< Accept: */*
< Connection: keep-alive
<
> HTTP/1.1 200 go to root plus 647174bf931a2fc29f11e44b66c903d04960778e
> Server: nginx/1.6.2
> Date: Mon, 13 Mar 2017 06:55:10 GMT
> Content-Type: text/html; charset=utf-8
> Transfer-Encoding: chunked
> Connection: keep-alive
> Content-Encoding: gzip
{% endhighlight %}

Srly?! I hate guests, get out!

Строка `"go to root plus 647174bf931a2fc29f11e44b66c903d04960778e"` явно указывает, что надо попробовать перейти на http://deep.ructf.org/647174bf931a2fc29f11e44b66c903d04960778e. Однако сайт не пускает нас, сообщая о том что наш браузер забанен. Перебирая различные значения User-Agent приходим к выводу, что он должен быть случайным, иначе сайт нас запомнит и не пустит второй раз с тем же User-Agent'ом. В ответе сервера также присутствует сообщение "go to root plus ...", но уже с другим хексом, поэтому логичным шагом будет автоматизировать генерацию User-Agent'a и переход на следующую ссылку.

Скрипт на третьем питоне (с использованием requests и requests_toolbelt):

{% highlight python %}
import requests, re, string, random
from requests_toolbelt.utils import dump

def gen_str(N):
  return ''.join([random.choice(string.ascii_letters) for _ in range(N)])

next_url = ""

while True:
  r = requests.get("http://deep.ructf.org/"+next_url, headers={"User-Agent": gen_str(20)})
  raw = dump.dump_all(r).decode()
  print(raw)
  next_url = re.findall("plus ([^\r\n]+)", raw)[0]
{% endhighlight %}
