---
title: Python读写pdf：茫茫辞海里，寻找我想要的你
date: 2019-12-9
author: Huiyu
summary: Coding for Love and Interesting！
categories: Tools
tags: 
 - Python
 - PDF
---
### 缘起
需要从某个顶会的pdf论文集中寻找和自己研究方向相关的论文，但是挨个pdf去ctrl+F太浪费生命了。我们尽量用最快捷的办法，解决低秩繁琐的事情，能让程序做的事情，拒绝暴力手工。
流程：按照关键词搜索并保存相关页面，这样，我可以顺便读一下摘要，筛选真正想读的论文。
## 安装依赖包
pdfminer3k读取pdf
~~~python
pip install pdfminer3k
~~~
PyPDF2保存pdf
~~~python
pip install PyPDF2
~~~
## 走起我的小程序
~~~python
from pdfminer.pdfinterp import PDFResourceManager, PDFPageInterpreter
from pdfminer.converter import PDFPageAggregator
from pdfminer.layout import LAParams
from pdfminer.pdfdevice import PDFDevice
from pdfminer.pdfparser import PDFParser, PDFDocument
from PyPDF2 import PdfFileReader, PdfFileWriter
import re
import os
import time

def PDFseeker(fileName,savedName,keyWord):
    pattern = re.compile(keyWord)#解析目标关键词
    fp=open(fileName,"rb")
    parser=PDFParser(fp)#创建一个与文档关联的解释器
    doc=PDFDocument()#PDf文档的对象
    resource=PDFResourceManager()#创建PDF资源管理器
    laparam=LAParams()#参数分析器
    device=PDFPageAggregator(resource,laparams=laparam)#创建一个聚合器
    interpreter=PDFPageInterpreter(resource,device)#创建PDF页面解释器
    #链接解释器和文档对象
    parser.set_document(doc)
    doc.set_parser(parser)
    #抓取目标页面
    pageindex = []
    i = 0
    for page in doc.get_pages():#处理每一页
        interpreter.process_page(page)#使用页面解释器来读取
        layout = device.get_result()#使用聚合器来获得内容
        for out in layout:
            if hasattr(out, 'get_text'):#鉴于PDF既有text也有图片等等，为了确保不出错先判断对象是否具有 get_text()方法
                if pattern.search(out.get_text()):
                    pageindex.append(i)
        i += 1
    #保存目标页面
    if len(pageindex) > 0:
        pdfWriter = PdfFileWriter()
        pdfReader = PdfFileReader(fp)
        for j in pageindex:#获取pdf共用多少页
            pdfWriter.addPage(pdfReader.getPage(j)) #将一个 PageObject 加入到 PdfFileWriter
        final_path = os.path.join(savedName)
        with open(final_path, "wb") as f:
            pdfWriter.write(f)
    else:
        print(keyWord,'is not found in',fileName)
    fp.close()

if __name__ == '__main__':
    start_time = time.time()

    fileName = "./Test.pdf"  
    savedName = "./final.pdf" 
    keyWord = "Huiyu Li" 
    PDFseeker(fileName, savedName, keyWord)

    print('Time {:.3f} min'.format((time.time() - start_time) / 60))
    print(time.strftime('%Y/%m/%d-%H:%M:%S', time.localtime()))
~~~

