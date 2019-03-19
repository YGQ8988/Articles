---
title: Python爬取桌酷壁纸站高清壁纸
date: 2018-08-10 17:07:59
tags:
---
2017-10-07  更新
==========
1：国庆没出去耍，就想着更新更新之前写的文章
---------
这次代码跟第一次比真的是改了很多，全部写了异常处理，以及多进程。

2：爬取思路
----------
这网站确实够折腾。首先从index页面获取所有专辑链接，然后再从专辑链接里面获取所有图片链接，最后再访问图片链接获取原图链接，再最后就是访问原图链接进行保存了。（这里是一个页面的思路，一个思路走通了，后面for循环下就行了！）

3：运行环境
----------
我这里每个专辑里面只处理了第一页的数据，第二页没去构造怎么去获取，有兴趣的朋友可以去试试。
```
# Python3
from bs4 import BeautifulSoup
import requests
import re
import os
#配置变量文件
from Set import *  
import multiprocessing
```
4：配置文件内容
----------
此内容需单独保存一个py文件。文件名为：Set.py
大家也可以修改，只需在程序里面改下引用的文件名。
如果IP代理不能用大家网上随便搜一个替换下。
```
#访问请求头
HEADERS = {'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36'}
#专辑URL前段
FIRST = 'http://www.zhuoku.com'
#专辑每张图片URL前段
FENGJING = 'http://www.zhuoku.com/zhuomianbizhi/'
#URL后缀
SLASH = '/'
#目录后缀
BACKSLASH = '\\'
#存储初始目录
PATH = 'D:\\bizhi\\'
#IP代理
PROEX = {'https':"https://111.13.7.120:80"}
```
5：程序全部代码
----------
如果大家想爬取其他table下图片，只需把程序里main函数内，initialurl参数替换下就行了。

![主页.png](http://upload-images.jianshu.io/upload_images/5324027-103dc02c8bbaebe9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我这里说明一下，由于启用了多进程，所以这个张数会出现重复。因为每个进程都是同时运行的。
![运行时输出信息.png](http://upload-images.jianshu.io/upload_images/5324027-c37ffb4e665434b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
from bs4 import BeautifulSoup
import requests
import re
import os
from Set import *
import multiprocessing

def getHtml(url):
    '''获取HTML文本'''
    try:
        html = requests.get(url,headers=HEADERS,timeout=15,proxies=PROEX)
        html.encoding = html.apparent_encoding
        html.raise_for_status()
        return html.text
    except BaseException as f:
        print('获取HTML文本失败，错误信息为：',f)

def geturl(html,seting):
    '''获取专辑以及每张图片URL'''
    try:
        soup = BeautifulSoup(html,'lxml')
        urlhtml = soup.find_all('div',class_='bizhiin')
        patten = re.compile('<a href="(.*?)" target="_blank">')
        urllist = re.findall(patten,str(urlhtml))
        for url_b in urllist:
            url = seting + url_b
            yield url
    except BaseException as f:
        print('获取专辑链接失败，错误信息为：',f)

def getimg(url):
    '''获取原图URL'''
    try:
        html = getHtml(url)
        soup = BeautifulSoup(html,'lxml')
        imgs = soup.find('div',id='bizhiimg')
        patten = re.compile('![]((.*?))')
        imgurl = re.findall(patten,str(imgs))
        return imgurl[0]
    except BaseException as f:
        print('获取图片真实URL失败，错误信息为：',f)

def gettitle(url):
    '''获取专辑标题'''
    try:
        html = getHtml(url)
        soup = BeautifulSoup(html, 'lxml')
        title = soup.find('div',id='liebiaom')
        return (title.h1.get_text().split(' ')[0])
    except BaseException as f:
        print('获取专辑链接标题失败，错误信息为：',f)

def spath(title):
    '''创建保存目录'''
    try:
        first_path = PATH
        if not os.path.isdir(first_path):
            os.mkdir(first_path)
        save = PATH+title+BACKSLASH
        if not os.path.isdir(save):
            os.mkdir(save)
        return save
    except BaseException as f:
        print('创建目录失败，错误信息为：',f)

def savaimg(initurl,url,path):
    '''保存图片'''
    try:
        headers = {
                    "Host": "bizhi.zhuoku.com",
                   "User-Agent": "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:56.0) Gecko/20100101 Firefox/56.0",
                   "Referer": initurl
                   }
        filename = url.split('/')[-1]
        file = open(path+filename,'wb')
        r = requests.get(url,headers=headers,timeout=15,proxies=PROEX).content
        file.write(r)
        file.close()
    except BaseException as f:
        print('保存图片失败，URL:{},错误信息为：'.format(url),f)
        
def main(page):
    '''主程序'''
    try:
        initialurl = 'http://www.zhuoku.com/zhuomianbizhi/show/index-{}.htm'.format(page)
        html = getHtml(initialurl)
        num = 0
        for url in geturl(html, FIRST):
            iurl = url.split('/', 5)
            html1 = getHtml(url)
            title = gettitle(url)
            savepath = spath(title)
            print('-----正在保存{}专辑链接-----'.format(title))
            for surl in geturl(html1, FENGJING + iurl[4] + SLASH):
                imgurl = getimg(surl)
                savaimg(url, imgurl, savepath)
                num += 1
                print('-----专辑链接：{}，成功保存{}张图片-----'.format(url,num))
    except BaseException as f:
        print('主程序运行出错，错误信息为：',f)

if __name__ == '__main__':
    '''多进程处理'''
    threads = []
    #这里的就是所有index页数，大家可以去页面看总页数在这里填上（加1，否则最后一页爬不到）。
    for page in range(1,36):
        t = multiprocessing.Process(target=main,args=(page,))
        threads.append(t)
    for a in range(len(threads)):
        threads[a].start()
```
2017-06-22  第一次编辑
==========
本人也是半道学习的Python。第一次成功爬取网站的图片。比较激动。特此发帖纪录。

欢迎大佬指导，纠错，谢谢！

运行环境为Python3（使用到的模块有，requests，bs4，re，os，hashlib）

此代码中演示的为桌酷壁纸站下的明星壁纸Table下图片。（需要爬取其他table下请自行替换）

先给大家看下我放服务器爬的30页数据成果吧。7560张，2.1G。

![image.png](http://upload-images.jianshu.io/upload_images/5324027-25611a618f2d63a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


考虑到，复制比较麻烦，附上链接（http://pan.baidu.com/s/1i4XZakd）
废话不多说，贴代码。本人比较菜，代码中没有写异常处理。
```
from bs4 import BeautifulSoup #导入bs4解析库
import requests,re,os,hashlib #导入全部写在这一行里面了
headers = {'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36'}
#设置浏览器User-Agent（模拟浏览器访问）

def urls1():   #创建获取页面专辑链接地址
	nums1 = int(input('请输入你要爬取的开始页数：'))
	nums2 = int(input('请输入你要爬取的结束页数：'))
	urls = []   #设置一个空列表，获取后加入到该列表中
	for num in range(nums1,nums2):
		url = 'http://www.zhuoku.com/zhuomianbizhi/star/index-'+str(num)+'.htm'
        #依次访问输入开始至结束的页面
		html = requests.get(url=url,headers=headers)
		html.encoding = 'gb2312' #设置编码，防止获取的页面文本乱码
		soup = BeautifulSoup(html.text,'lxml')
		for a in soup.select('.bizhiin'):
			b = a.select('a')[0]['href'] #获取 div 下bizhiin里所有链接
			c = 'http://www.zhuoku.com'+b  #由于获取到的不是完整的，这里拼接了下
			print('正在获取需要下载的地址链接:',c)
			urls.append(c)
	return urls   #返回需要下载的地址列表，以便后面函数调用
def pathdir():
	e = 'D:\\bizhi'
	if not os.path.isdir(e): #判断目录是否存在，不存在则创建
		print('-----不存在该目录-----')
		print('-----正在创建该目录-----')
		os.makedirs(e)  #使用os模块在本地创建目录
		print('-----目录创建成功-----')
	else:
		print('-----该目录已存在-----')
	return e  #返回创建的目录，后面保存用。
def urls2(urls,e):  #创建获取高清壁纸函数
	photos = []  #创建空列表，后续存放高清壁纸链接
	number = 0  
	for urlss in urls:  #迭代上面返回的专辑链接
		for x in range(1,6):  #只保存一个专辑内5张图片
			y = urlss.split('.')  #将获取的链接切片
			z = y[2]+'('+str(x)+')'+'.'+y[3] #拼接成高清图片后缀链接
			r = 'http://www.zhuoku.'+z    #拼接成完整链接
			print('正在获取专辑汇总链接：',r)
			pic = requests.get(url=r,headers=headers)
			pics = re.findall('http://bizhi.zhuoku.com.*?\.jpg',pic.text) #用RE匹配高清图片下载地址
			for photo in pics:
				if photo not in photos:
					photos.append(photo)  #去重处理
	for pict in photos:
		pices = requests.get(pict).content  #已content模式访问图片地址
		filename = hashlib.md5(pices).hexdigest()  #获取图片MD5值，作为文件名
		with open(e+'\\'+str(filename)+'.jpg','wb')as f:  #保存图片
			number = number + 1
			print('成功保存%d张图片：'% number,pict)
			f.write(pices)
			f.close()
	print('-----全部下载成功-----')

urls2(urls1(),pathdir())  #调用保存图片函数
```
