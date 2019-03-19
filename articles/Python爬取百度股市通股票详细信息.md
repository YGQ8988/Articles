---
title: Python爬取百度股市通股票详细信息
date: 2018-08-10 17:10:33
tags:
---
# 爬取思路来源
最近一直在看北京理工大学嵩教授主讲的：Python网络爬虫与信息提取
课程里面也有很多实例。大家可以去中国MOOC搜索查看下实例。
不过教授的思路挺好的。代码稳定性也没得说。全程都要异常处理。
今天记录的爬取百度股市通代码。我首先看了下教授的爬取思路，然后自己开始写代码。
在股票获取详细信息的时候搞不定了，借鉴了下老师的代码，最终拿下那些数据。
然后继续下一步。
最后成功保存到了股票详细信息。
最后去看了下老师的完整代码。看到老师写了进度，我也就借鉴了下。
这里我有几处自以为挺好，哈哈。
# 老师思路
老师获取股票代码先通过bs4解析，在正则匹配a标签里面的数据。
老师保存的股票信息也只是纯保存了获取到后存在字典里的数据。
# 我的思路
股票代码直接用正则去匹配，省去了bs4这步。
保存数据这块我也去遍历了字典取出键和值保存。（感觉美观一点）
# 运行
第一次写完运行发现进度太慢了。
今天看了下threading模块。（理解能力比较差，看了半天只会这一点。）
直接开启了五个线程执行。下面的图片可以看到执行了50分钟才跑完。
队列还不会，这里就不说了。还望大佬们指点，评论。谢谢！

![跑完后截得图.png](http://upload-images.jianshu.io/upload_images/5324027-a0c16943ad3f567b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
下面大家看代码吧。代码应该还能优化，能力有限，就写到这里了。

![文本内容.png](http://upload-images.jianshu.io/upload_images/5324027-6b331da106112ec8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
import requests
from bs4 import BeautifulSoup
import re
import bs4
import os
import threading


headers = {'User-Agent':
               'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36'
           }


def getHtmltext(url):
    '''获取东方财富，百度股市通股票详情页面源码'''
    try:
        r = requests.get(url,headers=headers)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
    except:
        print('-----获取股票代码链接打开失败-----')

    return r.text


def getstocklist(html):
    '''获取东方财富网沪深所有股票代码'''
    slist = []
    try:
        pat = r'http://quote.eastmoney.com/s[hz]\d{6}\.html'
        s_list = re.findall(pat,html)
        for stock in s_list:
            stock = stock.split('/')[-1]
            slist.append(stock)
    except:
        print('-----股票代码匹配出错了-----')

    return slist


def getbainfo(urls):
    '''获取股票详细信息'''
    count = 0
    for url in urls:
        url = 'https://gupiao.baidu.com/stock/'+url
        try:
            count+=1
            stock = {}
            html = getHtmltext(url)
            soup = BeautifulSoup(html,'lxml')
            stockinfo = soup.find('div',class_='stock-bets')
            title = stockinfo.text.split()[0]
            stock.update({'股票链接':url})
            stock.update({'股票名称':title})
            keylist = stockinfo.find_all('dt')
            vallist = stockinfo.find_all('dd')
            for i in range(len(keylist)):
                key = keylist[i].text
                val = vallist[i].text
                stock[key] = val
            #调用保存信息函数
            saveinfo(stock,count)
        except:
            print('-----该股票在百度股市通未收录-----'+url)


def saveinfo(stock,count):
    '''保存股票信息'''
    path = "D:/baidu/"
    if not os.path.exists(path):
        os.mkdir(path)
    try:
        print('正在保存{}股票信息,当前保存进度为{:.2f}%'.format(stock['股票名称'],count*100/ 898))
        for key,values in stock.items():
            with open(path+'Gupiao.txt','a')as f:
                f.write(key + ':')
                f.write(values + '\t')
                f.close()
        with open(path+'Gupiao.txt','a')as file:
            file.write('\n\n')
            file.close()
    except:
        print('提示：未收录股票无相关信息，当前保存进度为{:.2f}%'.format(count * 100 / 898))


def main():
    '''程序入口'''
    stock_url = 'http://quote.eastmoney.com/stocklist.html'
    html = getHtmltext(stock_url)
    urllist = getstocklist(html)
    #股票代码列表长度为：4487 print(len(urllist))
    #给线程分工，分别执行列表里哪些数据
    ta = threading.Thread(target=getbainfo,args=(urllist[:898],))
    tb = threading.Thread(target=getbainfo,args=(urllist[898:1796],))
    tc = threading.Thread(target=getbainfo, args=(urllist[1796:2694],))
    td = threading.Thread(target=getbainfo, args=(urllist[2694:3592],))
    te = threading.Thread(target=getbainfo, args=(urllist[3592:4487],))
    #开始线程
    ta.start()
    tb.start()
    tc.start()
    td.start()
    te.start()
if __name__ == '__main__':
    main()
```

![运行时输出信息.png](http://upload-images.jianshu.io/upload_images/5324027-ddb32f539bc5744a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
