# 连续请求某只股票tick数据并生成csv文件(Python 3.6)

## 1.导入模块

导入程序中需要的模块.

```python
from QuantBaseApi import QuantCallBack
from QuantBaseApi import PT_QuantApi_Python36
import time
import copy
import csv 
```

## 2.创建交易回调类

创建**交易回调类**，继承父类的回调方法，用于接收回调信息。比如，登录成功后，用户会接收到对应的登录回调信息；订阅行情成功后，会收到对应的订阅行情回调信息。

```python
class DataCallBack(QuantCallBack):
    
    def __init__(self):
        super(DataCallBack, self).__init__()
    def OnConnect(self, type):
        print("OnConnnect:", type)
        #type值的含义：1为实时服务器 2为历史服务器 3为缓存服务器  推荐在1.2.3三种行情服务器全部连接成功后做行情请求能保证全时段行情稳定回调。10为交易服务器，连接成功之后才可做交易业务请求
        pass
    def OnDisconnect(self, type):
        print("OnDisconnect:", type)
        #type值的含义同上
        pass
    def OnRtnUserInfo(self, pInfo):           # 用户基本信息
        print ("OnRtnUserInfo", pInfo)
        pass
    def OnRtnDayBegin(self, nReqId, pDate):	  # 通知开市消息
        print ("OnRtnDayBegin",pDate)
        pass
    def OnRtnDayEnd(self, nReqId, pDate):     # 通知闭市消息
        print ("OnRtnDayEnd",pDate)
        pass
    def OnRspSubQuote(self, pData):           # 订阅行情响应，行情数据推送结束后触发
        print ("OnRspSubQuote",pData)
        pass
    def OnRtnOrderQueue(self, pOrderQueue):   # 通知逐笔成交
        print ("OnRtnOrderQueue",pOrderQueue)
        pass
    def OnRtnMarket(self, pMarket):           # 通知个股行情
        print ("OnRtnMarket",pMarket)
        pass

```

## 3.创建连接对象实例

根据创建的交易回调类，创建**连接对象实例**。

```python
mspi = DataCallBack()；      #创建回调类
mapi = PT_QuantApi_Python36.PT_QuantApi(mspi, True, "Td_Real", "MD_Real")  #此处设置，创建的是交易的实盘环境，以及行情的实时环境    交易模拟环境在账号有权限的情况下将Td_Real替换成Td_SimulateTd即可
PT_QuantApi_Python36.Init()
```

## 4.用户登录

根据已创建的连接对象实例，调用**登录函数**，服务端校验用户名、用户密码的合法性。

用户登录后，触发用户登录回调，回调函数接收相应的回调信息。

```python
retLog = mapi.Login("DevTest1", "abcd1234")
print("QuantPlus Login :", retLog)#打印登录返回码
```

## 5.订阅行情参数赋值

### 5.1 操作流程

用户登录成功后，若需**订阅行情**，则需先对之进行**参数赋值**操作。

### 5.2 参数详解

nReqId：reqid

SubType：订阅行情类型

CycType：K线周期类型

pSubWindCode：请求的代码表

szBeginTime：指定起始时间

szEndTime：指定结束时间

```python
 nReqId = 1                              # 整型
 SubType = ["order_queue","market"]      # 列表
 CycType = [""]                          # 列表
 pSubWindCode = ["002003.SZ"]            # 列表，空值表示请求全市场代码
 szBeginTime = "2017-07-24 8:30:00"      # 字符串
 szEndTime = "2017-07-24 24:00:00"       # 字符串
```

## 6.调用订阅行情数据接口

将赋值后的参数放进**订阅行情接口**中，**API内部**会依据参数值对请求进行处理。

处理完成后，系统触发通知相关回调函数，回调函数收到相应的行情信息。

```python
mapi.ReqSubQuote(nReqID, SubType, CycType, pSubWindCode, szBeginTime, szEndTime)
```

## 7.Demo

```python
#encoding:utf-8

##
##
from QuantBaseApi import QuantCallBack
from QuantBaseApi import PT_QuantApi_Python36
import time
import copy
import csv 


class DataCallBack(QuantCallBack):
    
    def __init__(self):
        super(DataCallBack, self).__init__()
        self.api = PT_QuantApi_Python36.PT_QuantApi(self, True, "Td_Real", "MD_Real")
        self.listMarket=[]
        self.listOrderQueue=[]
    def OnConnect(self, type):
        print("OnConnnect type :", type)
        pass
    def OnDisconnect(self, type):
        print("OnDisconnect:", type)
        pass
    def OnRtnUserInfo(self, pInfo):
        pass
    def OnRspOrderInsert(self, pRsp, nErr):
        pass
    def OnRspOrderDelete(self, pRsp, nErr):
        pass
    def OnRspQryOrder(self, pRsp, nErr, isEnd):
        print ("OnRspQryOrder",pRsp)
        pass
    def OnRspQryMatch(self, pRsp, nErr, isEnd):
        pass
    def OnRspQryPosition(self, pRsp, nErr, isEnd):
        pass
    def OnRspQryMaxEntrustCount(self, pRsp, nErr, isEnd):
        pass
    def OnRtnOrderStatusChangeNotice(self, pNotice):
        pass
    def OnRtnOrderMatchNotice(self, pNotice):
        pass
    def OnRtnUserAuthen(self, pNotice):
        pass
    def OnRspHaltingDay(self, pData):
        print ("OnRspHaltingDay",pData)
        pass
    def OnRspSubQuote(self, pData):
        print ("OnRspSubQuote",pData)

        #创建Market.csv文件
        Marketfile = open('Market.csv', 'w+',newline='')#newline=''说明是csv文件类型
        marketwriter=csv.writer(Marketfile)
        #写入表格头
        marketwriter.writerow(['行情类型','证券代码','时间','开盘价','最高价','最低价','涨停价','跌停价','成交总量','成交总金额'])
        for i in range(len(self.listMarket)):
            writedata=[self.listMarket[i]["szType"],self.listMarket[i]["szWindCode"],self.listMarket[i]["szDatetime"],self.listMarket[i]["nOpen"],self.listMarket[i]["nHigh"],self.listMarket[i]["nLow"],self.listMarket[i]["nHighLimited"],self.listMarket[i]["nLowLimited"],self.listMarket[i]["iVolume"],self.listMarket[i]["iTurnover"]]
            marketwriter.writerow(writedata)#写入文件
        Marketfile.close()#关闭文件
        ###############################################
        #创建orderqueue.csv文件
        OrderQueuefile = open('OrderQueue.csv', 'w+',newline='')#newline=''说明是csv文件类型
        OrderQueuewriter=csv.writer(OrderQueuefile)
        #写入表格头
        OrderQueuewriter.writerow(['行情类型','证券代码','时间','委托价格','买卖方向','挂单挡位'])
        for i in range(len(self.listOrderQueue)):
            writedata=[self.listOrderQueue[i]["szType"],self.listOrderQueue[i]["szWindCode"],self.listOrderQueue[i]["szDatetime"],self.listOrderQueue[i]["nPrice"],self.listOrderQueue[i]["szSide"],self.listOrderQueue[i]["nOrders"]]
            OrderQueuewriter.writerow(writedata)#写入文件
        OrderQueuefile.close()#关闭文件
        pass
    def OnRtnSimulationAccount(self, pData):
        pass        
    def OnRtnTradingCode(self, pWinCode, pOptionCode):
        pass
    def OnRtnTradingDay(self, pDay):
        pass
    def OnRtnHaltingDay(self, pDay):
        pass
    def OnRtnKLine(self, pKline):
        print ("OnRtnKLine",pKline)
        pass
    def OnRtnTransaction(self, pTransaction):
        pass
    def OnRtnOrderQueue(self, pOrderQueue):
        print ("OnRtnOrderQueue",pOrderQueue)
        tmp = copy.deepcopy(pOrderQueue)
        self.listOrderQueue.append(tmp)
        pass
    def OnRtnOrder(self, pOrder):
        pass
    def OnRtnDayBegin(self, nReqId, pDate):
        print("OnRtnDayBegin",pDate)
        pass
    def OnRtnDayEnd(self, nReqId, pDate):
        print ("OnRtnDayEnd",pDate)
        pass
    def OnRtnMarket(self, pMarket):
        print ("OnRtnMarket",pMarket)
        tmp = copy.deepcopy(pMarket)
        self.listMarket.append(tmp)
        pass
    def OnRspQryAccountMaxEntrustCount(self,pRsp, nErr, isEnd):
        pass
    def OnRtnMaxEntrustCount(self,pNotice):
        pass
    def OnRspTradingDay(self,pData):
        pass
def main():
    PT_QuantApi_Python36.Init()
    mspi = DataCallBack()
    mapi = mspi.api

    mapi.Login("DevTest1", "abcd1234")

    time.sleep(1)    #此处休眠一秒，保证在行情服务器连接成功后做请求
    mapi.ReqSubQuote(1, ["order_queue","market"], [""], ["002003.SZ"], "2018-07-24 8:30:00", "2018-07-24 24:00:00")
    PT_QuantApi_Python36.Run()


if __name__ == '__main__':
    main()

```

