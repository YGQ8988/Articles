---
title: Python获取微信群网内个人，微信群二维码及账号
date: 2018-08-10 17:08:35
tags:
---
---------------------------------------------
# 一、介绍
---------------------------------------------
最近一直在写爬虫，前几天和老哥聊天。（老哥任职于某医疗公司）老哥也在研究爬虫。我就问他爬什么网站。然后老哥直接给我发了链接过来。我看了下：是个小网站。一看就挺好爬的。然后自己就写起代码来了。
网站为：http://www.weixinqun.com/。这个网站也真是绝了，各种行业个人，群信息都有。
有需要的也可以去试试。
这代码从周五晚上断断续续，一直写到今天上午才算真正调试完了。
晚上睡觉都在想出问题的地方改这么处理。
---------------------------------------------
# 二、爬取思路
---------------------------------------------
思路还是老套路。
用的模块我这里直接贴出来。
1.requests
2.bs43.xlwt
4.md5
5.os
也就是一些定向爬虫常用的一些模块。
由于数据比较多。这里采用的是excel保存相关信息，二维码单独存在标题目录下。
之前没看过xlwt模块，昨天晚上一直在研究这模块，官方示例代码很好理解。
QQ群里有朋友介绍用CVS保存。据他说比excel更简单。（后续我也去看看这模块）
代码里面除了主函数有6个函数。这里介绍下吧。
第一个函数：获取页面源码
第二个函数：通过一个函数获取到的源码，再获取每页的列表里数据详细链接
第三个函数：获取每个详细链接里面的数据
![圈中部分为需要获取的.png](http://upload-images.jianshu.io/upload_images/5324027-108b44e5c124b2d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
第四个函数：将获取到的写入excel文件
第五个函数：保存二维码图片至本地
第六个函数：将获取到的信息切片，方便存入excel。
![获取到的信息和页面展示一样，有冒号分开.png](http://upload-images.jianshu.io/upload_images/5324027-bcd4ee2f7e8d24f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![切片后存储方便查阅.png](http://upload-images.jianshu.io/upload_images/5324027-ab75540ac1d3dfa3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我比较懒，这里就不每个函数细介绍了，一会贴完整代码，代码里写了一些注释，大家自己看吧。
---------------------------------------------
# 三、代码自我评价
---------------------------------------------
1：这些代码优化空间应该还挺大的，比如写入excel的数据，我这里采用的是将所有的数据存到一个列表里面，然后再取出来写入excel文件。
2：这里我也没有写多线程。里面for写的有点多，写多线程怕搞懵了。（反正也是写着玩的，就没在意爬取效率了）（PS:其实我多线程还不怎么会用，哈哈）
3：我把代码丢到服务器爬了一个类型的所有个人微信号信息。爬完有（1162条信息）,二维码也都是以类似标题名称创建的文件夹下面。
---------------------------------------------
大家运行改下相应的存储路径吧。
---------------------------------------------
此代码请勿商用，如侵犯个人信息还请告知，本人立马删除！
---------------------------------------------
```
import requests
from bs4 import BeautifulSoup
import bs4
import xlwt
from hashlib import md5
import os


headers = {'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36'}

def gethtml(url):
    '''解析页面源码'''
    try:
        #添加浏览器头，访问超时
        html = requests.get(url,headers=headers,timeout=30)
        html.raise_for_status()
        html.encoding = html.apparent_encoding
        return html.text
    except:
        print('获取页面源码失败')


def getchaturls(html):
    '''获取每页信息详情链接'''
    try:
        soup = BeautifulSoup(html,'html.parser')
        chat_url = []
        for chat in soup.find_all('div',class_='border5'):
            chat = chat.a.attrs['href']
            chat_url.append('http://www.weixinqun.com'+chat)
        return chat_url
    except:
        print('获取页面详细详细链接失败')


def getinfo(url,infos):
    '''获取链接详情页面所需信息'''
    try:
        picurls = []
        html = gethtml(url)
        soup = BeautifulSoup(html,'lxml')
        pic = soup.find('div',class_='iframe')
        x = pic.find_all('span',class_='shiftcode')
        #判断是否详情页面是否有群主及群二维码
        if len(x) == 2:
            picurls.append(x[0].img.attrs['src'])
            picurls.append(x[1].img.attrs['src'])
        else:
            picurls.append(pic.img.attrs['src'])
        #找到详情页面需要数据所在的标签
        clearfix = soup.find('div',class_='des_info')
        #在需要的标签内获取标题
        title = clearfix.find('span',class_='des_info_text').get_text().replace('\n','').replace(' ','').replace(':','')
        #获取详细信息
        for info in clearfix.find('ul').children:
            if isinstance(info, bs4.element.Tag):
                other = info('li')
                for a in other:
                    infos.append(a.get_text().replace('\n','').replace(' ',''))
        #获取微信号
        waccount = clearfix.find_all('span',class_='des_info_text2')
        infos.append('微信号：'+waccount[1].get_text().replace('\n','').replace(' ',''))
        #获取热度
        hot = clearfix.find_all('span',class_='Pink')
        infos.append('热度：'+hot[0].get_text().replace('\n','').replace(' ',''))
        print('正常处理的详情标题：{}'.format(title))
        #调用保存二维码方法
        savepic(picurls,title)
    except:
        print('详情页面解析失败')


def saveinfo(infoms):
    '''保存excel文件'''
    wb = xlwt.Workbook()
    ws = wb.add_sheet('wchat')
    ws.write(0, 0, '行业')
    ws.write(0, 1, '时间')
    ws.write(0, 2, '地区')
    ws.write(0, 3, '标签')
    ws.write(0, 4, '微信号')
    ws.write(0, 5, '热度')
    pp = 1
    for b in range(0, len(infoms), 6):
        ws.write(pp, 0, infoms[b])
        ws.write(pp, 1, infoms[b + 1])
        ws.write(pp, 2, infoms[b + 2])
        ws.write(pp, 3, infoms[b + 3])
        ws.write(pp, 4, infoms[b + 4])
        ws.write(pp, 5, infoms[b + 5])
        pp += 1
        wb.save('D://微信群//wchat.xls')


def savepic(picurls,title):
    '''保存群或群主/个人微信二维码'''
    path = 'D://微信群//'
    if not os.path.exists(path):
        os.mkdir(path)
    path1 = path+title+'//'
    if not os.path.exists(path1):
        os.mkdir(path1)
    for url in picurls:
        photo = requests.get(url,headers=headers).content
        filename = md5(photo).hexdigest()
        with open(path1+filename+'.jpg','wb') as f:
            f.write(photo)
            f.close()


def infossplit(infos):
    '''字符串切片操作'''
    infoms = []
    for info in infos:
        info = info.split('：')[1]
        infoms.append(info)
    return infoms


def main():
    '''程序入口'''
    #创建空列表存放详情
    infos = []
    for page in range(0,13): #页面默认第一页为0，请至页面查看最后一页的数字，将最后一页数字加1输入括号第二位置
        #如需其他类型，请至浏览器查看对应类型链接的t值，如微信群链接开头为：http://www.weixinqun.com/group，个人链接开头为：http://www.weixinqun.com/personal
        first_url = 'http://www.weixinqun.com/personal?t=52&p={}'.format(page)
        print('正在处理第{}页，链接为：{}'.format(page,first_url))
        html = gethtml(first_url)
        urls = getchaturls(html)
        for url in urls:
            print('=====分隔符=====')
            print('正在处理第{}页，内容详情链接为：{}'.format(page,url))
            getinfo(url,infos)
    infoa = infossplit(infos)
    print(infos)
    saveinfo(infoa)
    print('=====全部处理完成=====')

if __name__=='__main__':
    main()
```
---------------------------------------------
