# freedriveJ - own your own data
数据上链和解析数据，真正让用户拥有自己的数据。freedriveJ是所有服务商提供的标准接口服务.
## 1. 规范说明  

### 1.1 通信协议
HTTPS协议   
请求路径: https://open.api.qingniao.cloud

### 1.2 请求方法
所有接口只支持POST方法发起请求。  

### 1.3 字符编码
接口metadata，data,是UTF-8字符编码的字节流。

### 1.4 签名规则

接口的签名,分user_signature 和 dapp_signature    
签名规则按如下计算:  
- 接口参数按顺序拼接      
  params_concat =  接口名称 + 参数1 + 参数2 ... + 时间戳
- hash = sha256(params_concat)
- user_signature是用user_addr的私钥签名hash  
  dapp_signature是用dapp_addr的私钥签名hash
- 时间戳timestamp过期时间为: 60s 

### 1.5 错误码
详情查看[errno](./errno.md)    

### 1.6 更新日志
详情查看[changelog](./changelog.md)    

### 1.7 接口使用案例
参看[demo](https://github.com/freedrivecash/test)


## 2. 接口定义
#### 常用
[2.1.  put](#put)  
[2.2.  get](#get)  
[2.3.  list](#list)   
#### 授权
[2.4.  get_drive_info](#get-drive-info)             
[2.5.  auth](#auth)     


### put   
- **接口说明:** 上链数据或更新数据  
drive_id有2种角色:admin和member,创建者默认是admin.  
可以调用接口[auth](#auth)进行设置角色

- **接口名称:** /api/put

#### 2.1.1 请求参数
  
参数名称				|类型		|出现要求	|描述  
:----					|:---		|:------	|:---	
&emsp;user_addr				|string		|Required		|用户地址
&emsp;dapp_addr				|string		|Required		|应用地址
&emsp;metadata				|string		|Required		|hex字节流
&emsp;data				|string		|Required		|hex字节流
&emsp;drive_id				|string		|Optional		|传"0",表示上链全新数据
&emsp;timestamp				|string		|Required		|时间戳
&emsp;user_signature			|string		|Required		|用user_addr签名的内容

#### 2.1.2 参数类型
```
["application/json"]  
```

#### 2.1.3 请求实列
```
curl http://open.api.qingniao.cloud/api/put  -X POST  -d @put.json  --header "Content-Type:application/json"
```

put.json内容如下
```
{
  "user_addr": "F9A9TgNE2ixYhQmEnB15BNYcEuCvZvzqxT", 
  "dapp_addr": "F9A9TgNE2ixYhQmEnB15BNYcEuCvZvzqxT", 
  "metadata":"61869fb46ccc915c36e2366d77ef8d", 
  "data": "01010101",
  "drive_id": "0"  
  "timestamp": "1593550887",
  "user_signature": "xxxxxxx" 
}   
```
#### 2.1.4 返回结果         
```
{
   "code":200,
   "msg":"",
   "result":"b5d8fd6e9393916b7f0355fc058039f56a3a0c9e2b884e4294e0df001f42a105"
}
```

### get 
- **接口说明：** 获取链上存储内容           
- **接口名称：** /api/get   

#### 2.2.1 请求参数
  
参数名称				|类型		|出现要求	|描述  
:----					|:---		|:------	|:---	
&emsp;dapp_addr				|string		|Required		|应用地址
&emsp;drive_id				|string		|Required		|传"0",表示上链全新数据
&emsp;prev				|int		|Required		|相对于drive_id向前索引
&emsp;next				|int		|Required		|相对于drive_id向后索引
&emsp;timestamp				|string		|Required		|时间戳
&emsp;dapp_signature			|string		|Required		|用dapp_addr签名的内容

#### 2.2.2 参数类型
```
["application/json"]  
```

#### 2.2.3 请求实列
```
curl http://open.api.qingniao.cloud/api/get  -X POST  -d @get.json  --header "Content-Type:application/json"

```
get.json内容： 
```
{
  "dapp_addr":"13LQro1Y3MQTwdoLJEL6ax55S4LNgWMUid",
  "drive_id": "ec08e062ba72a37aeb8e40ebbee5a7ba9b5db8f2965baabf69f1521e2d9f96e6",
  "prev": 10,
  "next": 10,
  "timestamp":"1596971994",
  "dapp_signature":"HxADTDsf3S8HH7swTFEVU7H9pL5wCOFg6L0l9IqkTqqQPfRiQglrHyY1ax15XjcDS3bRkeUDIwmwba2/gVKjSIc="
}
```			
#### 2.2.4 返回结果         
```
{
  "code": 200,
  "msg": "操作成功",
  "result": [{
    "drive_id": "18b5377090a6afbd09ee0c373fb66dc32e461eb399d50ed2c9c3276039e6b218",
    "metadata": "00",
    "data": "https://qingniao-block.oss-cn-hangzhou.aliyuncs.com/bbes/blockDrive/file/10f0c1f0-d12c-4a38-a6c5-d96b4e5f2d3b",
    "type": "1",
    "branch": "master",
    "prev": 1,
    "next": 0
    }, {
    "drive_id": "ec08e062ba72a37aeb8e40ebbee5a7ba9b5db8f2965baabf69f1521e2d9f96e6",  //  这个是传入的drive_id
    "metadata": "0101",
    "data": "https://qingniao-block.oss-cn-hangzhou.aliyuncs.com/bbes/blockDrive/file/9b8b21df-88ba-4c63-a2b2-c0543035dc04",
    "type": "1",
    "branch": "master",
    "prev": 0,
    "next": 0
    }, {
    "drive_id": "d03619fba0c5581e091cc0b53b38141dac67795ba725f9ea13431a9575a6dce0",
    "metadata": "383838389898",
    "data": "https://qingniao-block.oss-cn-hangzhou.aliyuncs.com/bbes/blockDrive/file/e89a9c28-c05c-48fa-bfd5-aaae06931f5a",
    "type": "1",
    "branch": "master",
    "prev": 0,
    "next": 1
    }, {
    "drive_id": "62255449801aa02fce02a3a4e8147023c9e9d0a214ae093f279ac73dfe942868",
    "metadata": "0101",
    "data": "https://qingniao-block.oss-cn-hangzhou.aliyuncs.com/bbes/blockDrive/file/b6875c85-12bd-4d99-a4e9-051e15ddf410",
    "type": "1",
    "branch": "master",
    "prev": 0,
    "next": 2
    }]
}

返回字段:
 type
    type = 1,data数据为链接
    type = 0,data数据为正常数据  

```
    
### list 

- **接口说明：** 搜索链上内容           
- **接口名称：** /api/list   

#### 2.3.1 请求参数
  
参数名称				|类型		|出现要求	|描述  
:----					|:---		|:------	|:---	
&emsp;protocol				|string		|Optional		|传"0"表示不指定协议查询
&emsp;addr				|string		|Optional		|传"0"表示不指定定制查询
&emsp;dapp_addr				|string		|Required		|应用地址
&emsp;page				|int		|Required		|查询第几页
&emsp;detail				|int		|Required		|(1:true, 表示获取drive_id详情;  0:false, 表示不返回driveid详情)
&emsp;timestamp				|string		|Required		|时间戳
&emsp;dapp_signature			|string		|Required		|用dapp_addr签名的内容

#### 2.3.2 参数类型
```
["application/json"]  
```
#### 2.3.3 请求实列
```
curl http://open.api.qingniao.cloud/api/list  -X POST  -d @list.json  --header "Content-Type:application/json"
```

list.json 内容：
```
detail:
   detail = 1,表明获取drive_id详情，只返回drive_id和update_id的最新1条数据
   detail = 0,表明只是返回drvei_id列表

参数类型: ["application/json"]    
{
  "protocol":"FOCP1V5",	 
  "addr":"0",  
  "dapp_addr":"13LQro1Y3MQTwdoLJEL6ax55S4LNgWMUid",
  "page": 1, 
  "detail": 1,   
  "timestamp":"1597002009",
  "dapp_signature":"HwluIXRPDvxDq/r9l95LMsS+xs37aFJcMUWmkUsHpAQaZhjsGe1yuvAKVlOVFvAh6E6dYI/MIF0LaVXqhI6dpPI="  
}
 ```
#### 2.3.4 返回结果           
```
当detail = 0时	    
返回结果：
{
   "code":200,
   "msg":"操作成功",
   "result": {
       "page":"2",
       "page_sum":"20"
       "drive_ids": ["f613da5785cfcfbb5c4d47e8dd11156712c8b9fa169881ec4c805ea4f6f1b6b6", "f613da5785cfcfbb5c4d47e8dd11156712c8b9fa169881ec4c805ea4f6f1b6b6"]	
    }
}

当detail = 1时
返回结果:
{
    "code": 200,
    "msg": "操作成功",
    "result": {
        "page": "1",
        "page_sum": "195",
        "drive_ids": [{
            "drive_id": "a8ac447e1e742f992eba67947ddbdfd2cde59d365fcb49152aba21bd454661aa",
            "metadata": "04464f435001310135064352454154454030354230334131324334414142414645423344304439463734413444454235454630334338353043303834464237324443434142423543314644303838303543000000057a682d434e00",
            "data": "https://qingniao-block.oss-cn-hangzhou.aliyuncs.com/bbes/blockDrive/file/0556b9af-b6c0-407b-a891-ca5a90c9f6b2",
            "type": "1"
        }, 
	...
	...		
	 {
            "drive_id": "a82c760c05217c98a274715a24c5b3921eae9df9ba3b1d6d60b45ba8c1563c08",
            "metadata": "04464f435001310135064352454154454036333946303332314339413130394141333637464333374231313434343630353043424638353543453133344542333332344443413446434242453430364131000000057a682d434e00",
            "data": "https://qingniao-block.oss-cn-hangzhou.aliyuncs.com/bbes/blockDrive/file/e8eb1206-00f6-490b-bdd0-ae153ec30e43",
            "type": "1"
        }]
    }
}
		  
```

### get drive info

- **接口说明：** 获取drive_id角色授权信息        
- **接口名称：** /api/get_drive_info   

#### 2.4.1 请求参数
  
参数名称				|类型		|出现要求	|描述  
:----					|:---		|:------	|:---	
&emsp;dapp_addr				|string		|Required		|应用地址
&emsp;drive_id				|string		|Required		|待查询的drive_id
&emsp;timestamp				|string		|Required		|时间戳
&emsp;dapp_signature			|string		|Required		|用dapp_addr签名的内容

#### 2.4.2 参数类型
```
["application/json"]  
```
#### 2.4.3 请求实列
```
curl http://open.api.qingniao.cloud/api/get_drive_info  -X POST  -d @get_drive.json  --header "Content-Type:application/json"
```

get_drive.json 内容：
```
{
  "dapp_addr":"1QrD3JVeeJxT56coCwCoPxi7Bm91unnyM"",
  "drive_id":"f4adf42047b18b7e8282cd17375c41bca7c166e5d72f27b50faaa57831ce",
  "timestamp":"1593550887",
  "dapp_signature":"xxxxxxxxxxx" 
}
```
#### 2.4.4 返回结果           
```
{
  "code": 200,
  "msg":"操作成功",
  "result": {
      "drive_id":"f4adf42047b18b7e8282cd17375c41bca7c166e5d72f27b50faaa57831ce",
      "prev": n, 
      "next": n, 
      "branch": "master",
      "admin":[addr1, addr2, ..., addrn],
      "member":[addr11, addr22, ..., addrn2]
  }
}
```
### auth 

- **接口说明：** 设置drive_id角色    
新创建的drive_id, 创建者默认是admin         
权限底层逻辑可参考[合约实现](https://qingniao-block.oss-cn-hangzhou.aliyuncs.com/bbes/blockDrive/file/0b53b21e-2c8d-4946-92c1-d7c656689a11.pdf)
- **接口名称：** /api/auth

#### 2.5.1 请求参数
  
参数名称				|类型		|出现要求	|描述  
:----					|:---		|:------	|:---	
&emsp;user_addr				|string		|Required		|用户地址
&emsp;dapp_addr				|string		|Required		|应用地址
&emsp;drive_id				|string		|Required		|待授权的drive_id
&emsp;admin				|array		|Required		|
&emsp;member				|array		|Required		|
&emsp;timestamp				|string		|Required		|时间戳
&emsp;user_signature			|string		|Required		|用user_addr签名的内容

#### 2.5.2 参数类型
```
["application/json"]  
```
#### 2.5.3 请求实列
```
curl http://open.api.qingniao.cloud/api/auth  -X POST  -d @auth.json  --header "Content-Type:application/json"
```

auth.json内容：
```
admin和member:
  都传0时,表明这个drive_id将不再被修改,处于终结状态
  拼接参数:取admin,member数组的元素进行拼接,不包含'[]'和','
admin:
  可以增删admin和member成员，包括自己 	
member:
  只能修改metadata,data内容，不能增删角色	

{
  "user_addr":"1QrD3JVeeJxT56coCwCoPxi7Bm91unnyM",	
  "dapp_addr":"1QrD3JVeeJxT56coCwCoPxi7Bm91unnyM",	
  "drive_id":"f4adf42047b18b7e8282cd17375c41bca7c166e5d72f27b50faaa57831ce",
  "admin":[addr1, addr2, ..., addrn],
  "member":[addr11, addr22, ..., addrn2],
  "timestamp":"1593550887",
  "user_signature":"xxxxxxxxxxxxx"  
}

或者admin,memeber传"0"表示终结drive_id,不能被修改
参数类型: ["application/json"]    
{
  "user_addr":"1QrD3JVeeJxT56coCwCoPxi7Bm91unnyM",	
  "dapp_addr":"1QrD3JVeeJxT56coCwCoPxi7Bm91unnyM",	
  "drive_id":"f4adf42047b18b7e8282cd17375c41bca7c166e5d72f27b50faaa57831ce",
  "admin":["0"],
  "member":["0"],
  "timestamp":"1593550887",
  "user_signature":"xxxxxxxxxxxxx" 
}
```
#### 2.5.4 返回结果           
```
{
   "code": 200,
   "msg": "操作成功",
   "result":"drive_id"
}
```



