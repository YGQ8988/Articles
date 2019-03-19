---
title: Python爬取电影网资源链接
date: 2018-08-10 17:10:03
tags:
---
# 一：介绍
其实我之前已经爬过这个网站，不过当时还是比较菜，获取数据方式比较复杂，也没有进行存储。有兴趣的可以看看我之前的简书。不想去更新那篇文章， 索性就直接重新发布一篇文章了。
在这里感谢企鹅群内，@皮皮瞎，指导，提供多线程处理方案。

# 二：使用环境
python：3.0+以上应该都没问题（我装的是3.6）
模块：requests，bs4，re，pymongo，gevent（没有的请先自行安装）

# 三：开始分析思路，敲代码
## 1：构造每页链接
其实还是老套路。链接：http://www.80s.tw/。
首先肯定是进入主页，然后再进入电影table下。然后爬取的初始链接就搞定了。链接：http://www.80s.tw/movie/list
分析每页链接，这里一进去默认就是第一页的数据。链接还是上面的。这里就暂时不去管它，让我们翻页观察下链接的变化。
![第二页链接](http://upload-images.jianshu.io/upload_images/5324027-e15d7d62e1314b89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![最后一页链接](http://upload-images.jianshu.io/upload_images/5324027-3d0bcbbf3cdcccec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通过观察第二页和最后一页的链接，就能发现规律，进行构成所有页数的链接了。
## 2：获取电影详情链接
下面我们看下每页列表下的超链接。
![每页电影详情.png](http://upload-images.jianshu.io/upload_images/5324027-4d0cdf83b8893cab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我这里说明下，大家也看到了，有两个a标签，而且有重复。（所以后面代码里面用了set(集合)去重）。
我们继续观察每个电影详情链接，
![电影详情链接](http://upload-images.jianshu.io/upload_images/5324027-a53c2a772e4323d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
就是拼接一个首页链接上去，就是每个电影详情页链接，其实这样方式的URL构造很常见，大家爬着爬着就会熟练了。
## 3：获取电影详情页面数据
我们点击去一个电影详情页面，鼠标放置“迅雷下载”右击下检查元素。观察页面结构。
![电影详情.png](http://upload-images.jianshu.io/upload_images/5324027-4ef049648719f11d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里我就不一一截图了，大家多点开几个电影详情，观察页面结构。
所有链接都会包含在class=‘dllist1’标签下面，而且进去默认都是停留在分辨率最高的table下面。这样就更好，我们不用再去处理，就能直接获取分辨率最高的迅雷，磁力链接。

![电影名称](http://upload-images.jianshu.io/upload_images/5324027-ad83c7f919fdd4f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
还是老方法，右击检查元素分析。所有电影详情页面的电影名称都存在class=‘font14w’标签下面。
## 4：分析总结
第一步：构造每页链接，获取每页链接HTML文本
第二步：通过每页链接HTML文本内获取每个电影详情链接，构造电影详情链接
第三步：通过电影详情链接获取电影标题，迅雷，磁力链接。
第四步：使用mongoDB进行数据存储（大家觉得麻烦可以去掉数据处理这块代码，改成文本保存也是可以的）
Windows下mongoDB配置请看我另一篇简书文章。
http://www.jianshu.com/p/404e1ae11b84
下面看下运行后的结果：
![mongoDB数据.png](http://upload-images.jianshu.io/upload_images/5324027-e11ada8faf996a5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
import requests
from bs4 import BeautifulSoup
import re
from pymongo import MongoClient
import gevent
from gevent import monkey

headers = {
			'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36',
			'Host':'www.80s.tw'
			}

def get_html(url):
	'''获取每页HTML文本'''
	try:
		html = requests.get(url,headers=headers,timeout=30)
		html.encoding = 'utf-8'
		html.raise_for_status()
		return html.text
	except BaseException as f:
		print('获取HTML文本失败，错误信息为',f)


def get_url(html):
	'''获取每页HTML文本内电影链接'''
	try:
		soup = BeautifulSoup(html,'lxml')
		urlslist = soup.find('ul',class_='me1 clearfix')
		patten = re.compile('/movie/\d+')
		urls = re.findall(patten,str(urlslist))
		for url in urls:
			url = 'http://www.80s.tw' + url
			yield url
	except BaseException as f:
		print('获取URL失败，错误信息为',f)


def get_info(url):
	'''获取每个电影链接内需要的数据'''
	try:
		date = {}
		html = get_html(url)
		soup = BeautifulSoup(html,'lxml')
		title = soup.find('h1',class_='font14w').text
		infos = soup.find('ul',class_='dllist1')
		patten = re.compile('href="(.*?)".+?src="(.*?)"',re.S)
		clist = re.findall(patten,str(infos))
		date['电影名称'] = title
		date['电影详情链接'] = url
		for a in set(clist):
			date['迅雷链接'] = a[0]
			date['磁力链接'] = a[1]
		return date
	except BaseException as f:
		print('获取下载链接出错了，错误信息为：',f)


def main(pages):
	'''程序入口'''
	try:
		#数据库连接
		conn = MongoClient('127.0.0.1', 27017)
		db = conn.movies
		movies = db.movies
		for page in pages:
			first_url = 'http://www.80s.tw/movie/list/-----p{}'.format(str(page))
			print('正常处理：{}'.format(first_url))
			html = get_html(first_url)
			for url in set(get_url(html)):
				print('正在保存链接：{}的数据'.format(url))
				date = get_info(url)
				movies.insert(date)
	except BaseException as f:
		print('主程序运行出错了，错误信息为：',f)

if __name__=='__main__':
	'''主程序'''
	urllist = list(range(402))
	#多线程处理
	monkey.patch_all()
	jobs=[]
	for x in range(41):
		job=None
		if x==0:
			job=gevent.spawn(main, urllist[1:(x+1)*10])
		elif 0 < x < 40:
			job=gevent.spawn(main, urllist[x*10+1:(x+1)*10])
		else:
			job=gevent.spawn(main, urllist[x*10+1:402])
		jobs.append(job)
	gevent.joinall(jobs)
	print('----------全部保存成功----------')
```
今天就到这里，后续再继续和大家分享 一点爬虫经验，喜欢的可以点个关注或者喜欢，谢谢！
