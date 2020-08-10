# 接口说明


# 1. 校验产品注册状态
# 2. 获取 & 校验Token
# 3. 内部产品登出接口
<br/>

## <b>1. 校验产品注册状态</b>

### <b>接口</b>  Product/CheckProductRegisterStateByHospital?hospitalCode=TLJSYY&productName=eWordPOD </b> ***Get***

|参数|说明|类型|
|---|---|---|
|hospitalCode|机构编码|string|
|productName|产品名称|string|


## <b>2. 获取 & 校验Token</b>
### 调用方式


> ###  一、 内部产品获取校验

eg： eWordIMCIS client => Server
## 调用过程
*  ```IMCIS``` 用 自身产品名称和机构编码到Token服务器获取token 


### 1. 获取

### 接口 /Token/RetriveInternal  ***Post***

```
{
  "ProductName":"eWordPOD",
  "HospitalCode":"TLJSYY",
  "RequestIP":"192.168.1.68"
}
```

### 2. 校验

### 接口 /Token/ValidateInternal ***Post***

header:

```
Authorization :"Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOiIyMDIwLzgvNiAxNzozOToyOSIsImV4cCI6MTU5NjcwNjk0OSwiaXNzIjoiZVdvcmQifQ.MLjQUM2q3M6ZBCUDLuoK7-6plIJ4omk42OclVnluLMU"
```
body:
```
{
  "ProductName":"eWordPOD",
  "HospitalCode":"TLJSYY",
  "RequestIP":"192.168.1.68"
}
```


> ###  二、url 方式获取校验

eg： eWordIMCIS => eWordViewer
## 调用过程
*  ```IMCIS``` 用 ```Viewer``` 的产品名称和机构编码到Token服务器获取token
*  ```IMCIS``` 将获取到的令牌 附带到 **Url** 上 访问Viewer
*  ```Viewer```收到请求后，取**url**上的Token, 并附带上```自身产品名称```和```机构编码```到**授权服务器**进行校验
### 1. 获取

### 接口 /Token/RetriveInternal   ***Post***

```
{
  "ProductName":"eWordViewer",//eWordViewer产品名称
  "HospitalCode":"TLJSYY", //eWordViewer所在机构
  "RequestIP":"192.168.1.68"
}
```

### 2. 校验

### 接口 /Token/ValidateInternal  ***Post***

header:

```
Authorization :"Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOiIyMDIwLzgvNiAxNzozOToyOSIsImV4cCI6MTU5NjcwNjk0OSwiaXNzIjoiZVdvcmQifQ.MLjQUM2q3M6ZBCUDLuoK7-6plIJ4omk42OclVnluLMU"
```
body:
```
{
  "ProductName":"eWordViewer",
  "HospitalCode":"TLJSYY",
  "RequestIP":"192.168.1.68"
}
```

> ###  三、产品接口互相调用

eg： eWordArchive -> eWordRIS
## 调用过程
*  ```eWordArchive``` 用 **自身产品名称**(```eWordArchive```)和**机构编码**到Token服务器获取token
*  ```eWordArchive``` 调用Ris接口， 并将获取到的令牌传给```eWordRIS```
*  ```eWordRIS``` 收到请求后携带**自身产品名称**(```eWordRIS```)和**机构编码** 以及收到的令牌到token 服务器校验

### 1. 获取（访问端完成）

### 接口 /Token/RetriveInteractive ***Post***

```
{
  "ProductName":"eWordArchive",//eWordArchive 产品名称
  "HospitalCode":"TLJSYY", //eWordArchive所在机构
}
```

### 2. 校验 （被访问端完成）


### 接口 /Token/ValidateInteractive  ***Post***


header:

```
Authorization :"Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOiIyMDIwLzgvNiAxNzozOToyOSIsImV4cCI6MTU5NjcwNjk0OSwiaXNzIjoiZVdvcmQifQ.MLjQUM2q3M6ZBCUDLuoK7-6plIJ4omk42OclVnluLMU"
```
body:
```
{
  "ProductName": "eWordRIS", //eWordRIS产品名称
  "HospitalCode":"TLJSYY"
}
```

> ### 四、其他第三方公司访问数据交换平台接口

eg： Other Pacs -> eWordDEP

## 调用过程
*  ```Other Pacs``` 再授权端注册后， 获取到```AppId```和```AppSecret```。
*  ```Other Pacs``` 用 **AppId** 、**Timestamp** 和 计算得到的**Sign** 访问```eWordDEP``` 获取令牌接口。

### 接口 /Token/RetriveExternal  ***Post***

eg：
* 第三方调用eWordDEP参数

``` 
{
  "Sign": "FFAD48038F74D8696AE63B885D541958",
  "Timestamp": "1596767814",
  "AppId": "ThirdPartID",
  ...//其他无关参数忽略
}
```
### sign 计算方式

```
//当前unix时间戳
var timestamp = new DateTimeOffset(DateTime.Now).ToUnixTimeSeconds();
//拼接appid appsecret 和timestamp
string txt = string.Contact(appId , appSecret, timestamp);
//对拼接结果进行md5  32 位大写 加密，
string sign = Md5(txt);
```

* eWordDEP 访问Token参数

此处 ```ProductName``` 和```HospitalCode```在 ```eWordDEP``` 处添加；

```
{
  "Sign": "FFAD48038F74D8696AE63B885D541958",
  "Timestamp": "1596767814",
  "AppId": "ThirdPartID",
  "ProductName":"eWordDEP", //此处第三方公司限定访问eWordDEP， 无法访问公司端其他产品
  "HospitalCode":"TLJSYY",
}
```

## <b>3. 内部产品登出接口</b>

eg： eWordArchive 用户登出


### 接口 /Token/RetriveInteractive ***Post***

```
{
  "ProductName":"eWordArchive",//eWordArchive 产品名称
  "HospitalCode":"TLJSYY", //eWordArchive所在机构
  "RequestIP":"192.168.1.68"//用户IP地址
}
```
