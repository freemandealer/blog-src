title: Python Tricks
date: 2015-08-26 13:08:37
tags: Note
---

> 开一篇博客作为Python备忘录

<!--more-->

## Python模拟浏览器登陆网站 ##
方式一，使用httplib2模块:[参考资料](http://blog.csdn.net/five3/article/details/7079140)

	http = httplib2.Http()
	myuri = 'http://www.2-vpn3.org/lr.action'
	mybody = {'user.nick': 'NAME', 'user.password': 'PASSWORD','validationCode': 'VCODE'} 
	myheaders = {'Content-type': 'application/x-www-form-urlencoded'}
	response, content = http.request(myuri, 'POST', headers=myheaders, 	body=urlencode(mybody))
	myheaders = {'Cookie': response['set-cookie']}
	myuri = 'http://www.2-vpn3.org/home!sl.action'
	response, content = http.request(uri=myuri, method='GET', headers=myheaders)

方式二，使用urllib2模块:

	import urllib2
	headers = {'User-Agent':'Mozilla/5.0 (Linux; U; Android 2.3.6; zh-cn; GT-S5660 Build/GINGERBREAD) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1 MicroMessenger/4.5.255'}
	req = urllib2.Request(url = 'http://dict.youdao.com/search?q=manipulate&keyfrom=dict.index', headers = headers)
	res = urllib2.urlopen(req)
	html = res.read()

> Tip: 可以使用Chrome的开发者模式下Network工具来分析数据包。

## Python正则表达式 ##

	ips = re.findall(r'\d+\.\d+\.\d+\.\d+', content) #寻找ip地址，返回列表

	# 判断record中是否包含日期 #
	if re.search('\[[0-9].[0-3]*[0-9]\]', record):
	    do_something()


## Python验证码识别 ##
安装依赖库:

	sudo apt-get install python-pil
	sudo apt-get install tesseract-ocr
	sudo pip install pytesserac

使用:

	import pytesseract
	from PIL import Image
	image = Image.open('vcode.jpg')
	vcode = pytesseract.image_to_string(image)