# freedriveJ - own your own data
java implementation freedrive, see [architecture](./freedrive.pdf)
### 目录
#### 常用

[1.  put](#put)  
[2.  get](#get)  
[3.  list](#list)   

#### 授权和分支

[1.  auth](#auth)     
[2.  get_drive_info](#get-drive-info)             
[3.  branch](#branch) (todo)    


#### 计费 (todo)    

[1.  get_balance](#get-balance)  
[2.  get_tx_history](#get-tx-history)


#### 迁移 (todo)

[1. get_sp_info](#get-sp_info)       
[2. test_drive_id](#test-drive_id)       
[3. migrate](#migrate)  


#### 其他 (todo)

[1.  archive](#archive)       
[2.  create_tx](#create-tx)       
[3.  proof_exist](#proof-exist)        

### 通用  
>URL: https://open.api.qingniao.cloud

接口类型和签名规则:    
```
所有接口都是post请求, 参数类型都是: ["application/json"]
所有接口的参数签名(signature)字段计算规则,分user_signature 和 dapp_signature
user_addr和dapp_addr可以是同一个地址,分别表示用用户地址和应用地址签名接口参数,按如下计算:

1) 每个接口参数拼接 params_concat =  接口名称 + 参数1 + 参数2 ... + 时间戳
2) hash = sha256(params_concat)
3) user_signature = 用user_addr的私钥签名hash  
   dapp_signature = 用dapp_addr的私钥签名hash
4) 时间戳timestamp过期时间为: 60s 
```

错误码:    
>详情查看[errno](./errno.md)    

计费说明: (每次请求即时扣除)
>详情查看[bill](./bill.md)

更新日志:    
>详情查看[changelog](./changelog.md)    

一. 常用

### put   
>存数据或者更新数据到freedrive.
>返回存储标识drive_id   
>drive_id有2种角色:admin和member     
>创建者默认是admin,可以调用接口[auth](#auth)进行设置角色     
>user_addr和dapp_addr可以相同     
>接口名称: /api/put
```
参数类型： ["application/json"]  
{
  "user_addr": "F9A9TgNE2ixYhQmEnB15BNYcEuCvZvzqxT", 
  "dapp_addr": "F9A9TgNE2ixYhQmEnB15BNYcEuCvZvzqxT", 
  "metadata":"61869fb46ccc915c36e2366d77ef8d", (hex 字符串)
  "data": "01010101",(hex 字符串),
  "drive_id": 0/"drive_id"  // 为0，表示新建，否则更新drive_id
  "timestamp": "1593550887",
  "user_signature": "xxxxxxx"  (用user_addr签名后的结果)
}   

返回结果         
{
   "code":0,
   "msg":"",
   "result":"b5d8fd6e9393916b7f0355fc058039f56a3a0c9e2b884e4294e0df001f42a105"
}
```

```
curl example: 
curl http://open.api.qingniao.cloud/api/put  -X POST  -d @put.json  --header "Content-Type:application/json"
```

### get 
>从freedrive获取存储内容        
>接口名称: /api/get   


```
参数类型: ["application/json"]  
实例:
{
  "dapp_addr":"F9A9TgNE2ixYhQmEnB15BNYcEuCvZvzqxT",
  "drive_id": "1f6dc4adf42047b18b7e8282cd17375c41bca7c166e5d72f27b50faaa57831ce",
  "prev": n, // 向前索引, n <= 100
  "next": n, // 向后索引, n <= 100
  "timestamp":"1593550887",
  "dapp_signature":"xxxxxxxx"	(用dapp_addr签名后的结果)
}
  
返回结果：
{
   "code":0,
   "msg":"",
   "result": [
        {
          "drive_id": "1f6dc4adf42047b18b7e8282cd17375c41bca7c166e5d72f27b50faaa57831ce"
          "metadata": {},
          "data": "http://xxx.xxx.xxx",
          "type": 1，
          "branch":master,
          "prev": 3,
          "next": 0
        },
        {
          "drive_id": "1f6dc4adf42047b18b7e8282cd17375c41bca7c166e5d72f27b50faaa57831ce"
          "metadata": {},
          "data": "0101",
          "type": 0,
          "branch":"master",
          "prev": 2,     
          "next": 0
        }
        {
          "drive_id": "1f6dc4adf42047b18b7e8282cd17375c41bca7c166e5d72f27b50faaa57831ce"
          "metadata": {},	
          "data": "http://xxx.xxx.xxx",
          "type": 1,
          "branch":"branch1",
          "prev": 1,
          "next": 0
        },
        {
          "drive_id": "1f6dc4adf42047b18b7e8282cd17375c41bca7c166e5d72f27b50faaa57831ce"
          "metadata": {},
          "data": "0101",
          "type": 0,
          "branch":"branch2",
          "prev": 0,
          "next": 0
        }
        {
          "drive_id": "1f6dc4adf42047b18b7e8282cd17375c41bca7c166e5d72f27b50faaa57831ce"
          "metadata": {},
          "data": "0101",
          "type": 0,
          "branch":"branch2",
          "prev": 0,
          "next": 1
        }
    ]
}

返回字段:
 type
    type = 1,data数据为链接
    type = 0,data数据为正常数据  

```
    
### list 
>用addr，protocol 过滤或者搜索储列表    
>接口名称: /api/list
```
detail:
   detail = 1,表明获取drive_id详情，只返回drive_id和update_id的最新1条数据
   detail = 0,表明只是返回drvei_id列表

参数类型: ["application/json"]    
{
  "protocol":"FOCP1V5",	 // 传"0"表示不指定某个协议查询
  "addr":"F8Z2aQkHkBFhb3GQfEWV7L88yMuApj7jMK",  // 传"0"表示不指定某个地址查询
  "dapp_addr":"F8Z2aQkHkBFhb3GQfEWV7L88yMuApj7jMK",
  "page": 1, // 获取第几页
  "detail": 0/1,   // (1:true, 表示获取drive_id详情;  0:false, 表示不返回driveid详情)
  "timestamp":"1593550887",
  "dapp_signature":"xxxxxxxxxxxxx"  (用dapp_addr签名后的结果)	
}
   
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
    "code":200,
    "msg":"操作成功",
    "result": {
        "page":"2",	
        "page_sum":"20",
        "drive_ids":[
            {
                "drive_id":"166637633731646661306530646230623731383065000000057a682d636e00",
                "metadata":"04464f435001310135063165643635366637633731646661306530646230623731383065000000057a682d636e00",
                "data":"036d63311fe9a1b6e9a1b6e98f91e5b195e5b8a6e69da5e79a84e7a7afe69e81e4bd9ce794a80000000000000000",
                "type":0
             },
             {
                "drive_id":"222166637633731646661306530646230623731383065000000057a682d636",
                "metadata":"04464f43500131013506443342443536313845463145383737324639394442000000057a682d434e00",
                "data":"http://xxx.com",
                "type":1
             }]
    }
}	     
		  
```

二. 计费

### get balance
>获取地址dapp_addr的积分余额     
>接口名称: /api/get_balance
```
参数类型: ["application/json"]    
{
  "dapp_addr":"F8Z2aQkHkBFhb3GQfEWV7L88yMuApj7jMK",
  "timestamp":"1593550887",
  "dapp_signature":"xxxxxxxxxxx"   (用dapp_addr签名后的结果)	
}


返回结果:
{
   "code": 200,
   "balance": 12345	
}
```


```
curl example:
curl http://freedrive.cash:8880/api/get_balance -X POST  -d @get_balance --header "Content-Type:application/json" ' 
```

### get tx history
>获取地址dapp_addr积分变更记录     
>接口名称: /api/get_tx_history
```
参数类型: ["application/json"]    
{
  "dapp_addr":"1QrD3JVeeJxT56coCwCoPxi7Bm91unnyM",
  "page": 12,
  "timestamp":"1593550887",
  "dapp_signature":"xxxxxxxxxxxx"  (用dapp_addr签名后的结果)		
}
	


返回结果:
{
   "code": 200,
   "page": 12,
   "page_sum": 123,
   "data":[{"type":"put", "change": -10, "timestamp":2020-06-17 19:09:02},
	   {"type":"update", "change":-10,"timestamp":2020-06-17 19:09:02},
	   {"type":"get", "change":-2,"timestamp":2020-06-17 19:09:02},
	   {"type":"list_drive_id", "change":-2,"timestamp":2020-06-17 19:09:02},
	   {"type":"charge", "change":100,"timestamp":2020-06-17 19:09:02},
	  ]
}
```


```
curl example:
curl http://freedrive.cash:8880/api/get_tx_history -X POST  -d @tx_history.json --header "Content-Type:application/json"
```

三. 分支&授权

### get drive info
>获取drive_id角色授权&分支信息     
>接口名称: /api/get_drive_info

```
参数类型: ["application/json"]    
{
  "dapp_addr":"1QrD3JVeeJxT56coCwCoPxi7Bm91unnyM"",
  "drive_id":"f4adf42047b18b7e8282cd17375c41bca7c166e5d72f27b50faaa57831ce",
  "timestamp":"1593550887",
  "dapp_signature":"xxxxxxxxxxx" (用dapp_addr签名后的结果)		
}

返回结果:
{
  "code": 200,
  "msg":"操作成功",
  "result": {
      "drive_id":"f4adf42047b18b7e8282cd17375c41bca7c166e5d72f27b50faaa57831ce",
      "prev": n, // 向前索引有几层
      "next": n, // 向后索引有几层
      "branch": "master",
      "admin":[addr1, addr2, ..., addrn],
      "member":[addr11, addr22, ..., addrn2]
  }
}
```
### auth 
>设置drive_id角色.    
>新创建的drive_id, 创建者默认是admin,只有admin角色     
>接口名称: /api/auth
```
admin和member:
  都传0时,表明这个drive_id将不再被修改,处于终结状态
  拼接参数:取admin,member数组的元素进行拼接,不包含'[]'和','
admin:
  可以增删admin和member成员，包括自己 	
member:
  只能修改metadata,data内容，不能增删角色	

参数类型: ["application/json"]    
{
  "user_addr":"1QrD3JVeeJxT56coCwCoPxi7Bm91unnyM",	
  "dapp_addr":"1QrD3JVeeJxT56coCwCoPxi7Bm91unnyM",	
  "drive_id":"f4adf42047b18b7e8282cd17375c41bca7c166e5d72f27b50faaa57831ce",
  "admin":[addr1, addr2, ..., addrn],
  "member":[addr11, addr22, ..., addrn2],
  "timestamp":"1593550887",
  "user_signature":"xxxxxxxxxxxxx"  (用user_addr签名后的结果)	
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
  "user_signature":"xxxxxxxxxxxxx"  (用user_addr签名后的结果)	
}

返回结果:
{
   "code": 200,
   "msg": "操作成功"
}
```



### branch
>/api/branch 设置分支
```
参数类型: ["application/json"]    
{
    "user_addr":"1QrD3JVeeJxT56coCwCoPxi7Bm91unnyM",
    "dapp_addr":"1QrD3JVeeJxT56coCwCoPxi7Bm91unnyM",
    "drive_id":"f4adf42047b18b7e8282cd17375c41bca7c166e5d72f27b50faaa57831ce",
    "branch": "branch_name",  // 不能填master
    "timestamp":"1593550887",
    "user_signature":"xxxxxxxxxxxxx" (用user_addr签名后的结果)
}
返回结果：
{
"code": 200,
"drive_id":
    [{
      "drive_id": "11111" ,
      "branch": "name"   
    },
    {
     "drive_id":"1111333"
      "branch":"master"  
    }]        
}

```


四. 迁移   

    
### get sp_info
>DAPP 获取SP的sp_info    
>接口名称: /api/get_sp_info
```
参数类型: ["application/json"]    
{
  "dapp_addr":"F8Z2aQkHkBFhb3GQfEWV7L88yMuApj7jMK",
  "timestamp":"1593550887",
  "dapp_signature":"xxxxxxxxxxxxx"	(用dapp_addr签名后的结果)
}
   
	    
返回结果：
{
   "code": 200,
   "sp_addr": "1xxxvcvsdfdf",
   "token_id" : "sdfdfsdffffffffffff 
}

```

    
### test drive_id
>测试drive_id是否包含sp_addr     
>接口名称: /api/test_drive_id
```
参数类型: ["application/json"]    
{
  "drive_id":"dsdfsdfsdffffffffffffffffff",	
  "dapp_addr":"F8Z2aQkHkBFhb3GQfEWV7L88yMuApj7jMK",
  "timestamp":"1593550887",
  "dapp_signature":"xxxxxxxxxxxxx"	(用dapp_addr签名后的结果)
}
   
	    
返回结果：
{
   "code": 200,
   "sp_addr":"11111111111111"
}

```


    
### migrate
>更换drive_id的sp_addr   
>接口名称: /api/migrate  
```
参数类型: ["application/json"]    
{
  "drive_id":"dsdfdfsdssssssssssfsdfsdf",	
  "user_addr":"F8Z2aQkHkBFhb3GQfEWV7L88yMuApj7jMK",
  "timestamp":"1593550887",
  "user_signature":"xxxxxxxxxxxxx"	(用user_addr签名后的结果)
}
   
	    
返回结果：
{
   "code": 200,
   "raw_tx": "hashdsfddddddddddddfsdf"
}

```



五. 其他
    
### archive
> 归档drive_id  
> 针对大文件,大于200M的文件  
> 接口名称: /api/archive   
```
参数类型: ["application/json"]    
{
  "drive_id":"dsdfdfsdssssssssssfsdfsdf",	
  "dapp_addr":"F8Z2aQkHkBFhb3GQfEWV7L88yMuApj7jMK",
  "timestamp":"1593550887",
  "dapp_signature":"xxxxxxxxxxxxx"	(用dapp_addr签名后的结果)
}
   
	    
返回结果：
{
   "code": 200
}
```

### create tx
> 创建交易   
> 接口名称: /api/create_tx

```
参数类型: ["application/json"]    
{
  "vout_script":"dsdfdfsdssssssssssfsdfsdf",	
  "metadata":"xxxx", (hex 字符串)
  "data": "abc1111", (hex 字符串)
  "dapp_addr":"F8Z2aQkHkBFhb3GQfEWV7L88yMuApj7jMK",
  "timestamp":"1593550887",
  "dapp_signature":"xxxxxxxxxxxxx"	(用dapp_addr签名后的结果)
}
   
	    
返回结果：
{
   "code": 200,
   "raw_tx":"xxxxxxxxxxxxx"
}
```


### proof exist
> drive_id 存在证明  
> 针对大文件,大于200M的文件  
> 接口名称: /api/proof_exist

```
参数类型: ["application/json"]    
{
  "drive_id":"dsdfdfsdssssssssssfsdfsdf",	
  "data_key":"xxxxxxxxxxxsdfsdf",
  "dapp_addr":"F8Z2aQkHkBFhb3GQfEWV7L88yMuApj7jMK",
  "signed_raw_tx":"xxxxxxxxxxxxx",
  "timestamp":"1593550887",
  "dapp_signature":"xxxxxxxxxxxxx"	(用dapp_addr签名后的结果)
}
   
	    
返回结果：
{
   "code": 200,
   "data_hmac":"xxxxxxxxxxxxx"
}
```
