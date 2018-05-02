---
layout: post
title: "Web Crawler"
date: 2016-02-19 21:27:47 +0800
comments: true
categories: python
---

Finished my first web scrawler ... =\_=|

Here is all about the spider. It's small but funny.

After I finish this web scrawler, I learn that. The most important thing to build a web scrawler is not the code but the inverse process. How to build your program like a web browser.

Static web page is easy to get but how about a dynamic web page. Nowadays, more and more website use `ajax` to meet their bussiness requirement. If you just send a simple request to the server of that web site, you will find than the returned page is just only part of the total user page that you saw in the web browser.

<!-- more -->

Here is my demonstration.
[duitang.com](www.duitang.com) is a funny images web site. I love a cute virtual character who is named as Mr.zhangcao . There is a lot of images of that guy. So I want build a web crawler to get that images all.

If you send a request to the [main page](http://www.duitang.com/album/56759353) and anlysis the returned page (Python's library **BeautifulSoup** is helpful to analysis html files), you will find there are some links in the html file. But you will only get about 20+ images but not all images that you viewed in the web browser. When you pull down the button of the brower, you will see more and more images coming out to user.

Why the returned page don't have all links of images? The answer is `ajax`(asynchronous JavaScript and XML) is a set of web development techniques using many web technologies on the client-side to create asynchronous Web application.

**The monitor in browswer will help programmer who is writing a web spider.**

Dig into the detail of http transport, you will find a special request which are sent to server by the client browser.

This is that special url.

``` python

http://www.duitang.com/napi/blog/list/by_album/?album_id=
&limit=24
&include_fields=top_comments%2Cis_root%2Csource_link%2Cbuyable%2Croot_id%2C
status%2Clike_count%2Csender%2Creply_count&start=48&_=1455881387791

```

If client request that url to the server, the server will return a json data object. The job left is to analysis the json object and then get the resource links.

The important part of building a web crawler is not the code but how to find the useful url and get more useful information.

Here is a screenshut for what my web scrawler got.
![images](/images/img_for_2016_02_19/screenshut.png)


``` python

"""
Programmer 	: EOF
E-mail 		: jasonleaster@163.com
File        : duitang.py
Date        : 2016.02.19

"""
#!/usr/bin/env python
# -*- coding: utf-8 -*-
 
import urllib
import urllib2
import json
import re
import time
 
downloadCount = 0


album_id = 72129639
limit = 100
start = 1

# This function return a URL and this URL will return a json object data.
def duitangURLMaker(album_id, limit, start):
    url_part_1 = "http://www.duitang.com/napi/blog/list/by_album/?album_id="
    url_part_2 = "&limit="
    url_part_3 = "&include_fields=top_comments%2Cis_root%2Csource_link%2Cbuyable%2Croot_id%2Cstatus%2Clike_count%2Csender%2Creply_count&start="
    url_part_4 = "&_=1455881387791"

    URL = url_part_1 + str(album_id) + url_part_2 + str(limit) + url_part_3 + str(start) + url_part_4

    return URL

def imgURLs(jsonDict):
	URLs = []
	object_list = jsonDict["data"]["object_list"]
	for item in object_list:
		URLs.append(item["photo"]["path"])

	return URLs

URL 		= duitangURLMaker(album_id, limit, start)
string 		= urllib.urlopen(URL).read()
jsonDict 	= json.loads(string)

URLs = imgURLs(jsonDict)

for url in URLs:
    Filename = "duitang_%s.jpeg" % (downloadCount)
    try:
        urllib.urlretrieve(url, Filename)
        print "Download from", url
        downloadCount += 1
    except:
        print "Analyse Failed"

```

### Download some pdf documents on a website


``` python

"""
Programmer 	:   EOF
E-mail 		:   jasonleaster@163.com
File        :   stanford.py
Date        :   2016.02.19

Description:
	This programmer will help user who is learn cs97 in stanford to download some pdf material in the website.

"""
import urllib
from BeautifulSoup import *
import urllib2
import json
import re
import time
 
URL = "http://web.stanford.edu/class/cs97si/"
html = urllib.urlopen(URL).read()
soup = BeautifulSoup(html)
tags = soup('a')

downloadCount = 0

pattern = ".pdf$"
regex = re.compile(pattern)
for tag in tags:
    fileName = tag["href"]
    url = URL + fileName
    
    """
    search(string[, pos[, endpos]]) --> match object or None.
    Scan through string looking for a match, and return a corresponding
    MatchObject instance. Return None if no position in the string matches.
    """
    url = regex.search(url)
    regex.search
    if url is not None:
        url = url.string
        try:
            urllib.urlretrieve(url, fileName)
            print "Download from", url
            downloadCount += 1
        except:
            print "Analyse Failed"
 

```


-------
Photo by Jason Leaster. LiuYe Lake in ChangDe, HuNan, China.
![images](/images/img_for_2016_02_19/liuyehu.jpg)
