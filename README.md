  * [基础知识](#基础知识)
     * [了解protocolbuffer3](#了解protocolbuffer3)
     * [websocket和http](#websocket和http)
     * [交易执行的基本过程](#交易执行的基本过程)
     * [试一试](#试一试)
  * [HTTP接口](#http接口)
     * [查询账号](#查询账号)
     * [查询交易](#查询交易)
     * [查询区块头](#查询区块头)
     * [提交交易](#提交交易)
  * [定义交易](#定义交易)
     * [操作码](#操作码)
     * [操作](#操作)
        * [创建账号](#创建账号)
        * [发行资产](#发行资产)
        * [设置metadata](#设置metadata)
        * [转移资产](#转移资产)
        * [设置权重](#设置权重)
  * [高级功能](#高级功能)
     * [控制权的分配](#控制权的分配)
     * [版本化控制](#版本化控制)
     * [表达式](#表达式)
     * [合约](#合约)
        * [内置函数](#内置函数)
           * [获取账号信息(不包含metada和资产)](#获取账号信息不包含metada和资产)
           * [获取某个账号的metadata信息](#获取某个账号的metadata信息)
           * [获取某个账号的资产信息](#获取某个账号的资产信息)
           * [获取区块信息](#获取区块信息)
           * [做交易](#做交易)
        * [内置变量](#内置变量)
           * [该合约账号的地址](#该合约账号的地址)
           * [调用者的地址](#调用者的地址)
           * [触发本次合约的交易](#触发本次合约的交易)
           * [触发本次合约调用的操作的序号](#触发本次合约调用的操作的序号)
           * [本次共识数据](#本次共识数据)
  * [错误码](#错误码)

## 基础知识
### 了解protocolbuffer3
布比区块链是用protocolbuffer3序列化数据的，protocolbuffer3是google推出的数据序列化协议，您如果不了解protocolbuffer3，请点击[这里](https://developers.google.com/protocol-buffers/docs/proto3)了解更多。
我们使用的所有数据格式都能在源码的```src\proto```目录中找到。其中chain.proto文件中定义的数据是和交易、区块、账号密切相关的。
###  websocket和http
布比区块链提供了websocket和http 两种API接口。您可以在 安装目录/config/bubi.json 文件种找到`"webserver"`和`"wsserver"`两个对象,它们指定了http服务端口和websocket服务端口。

```json
	"webserver": 
    {
		"listen_addresses": "0.0.0.0:29333",
		"remote_authorized": false
	},
	"wsserver": 
    {
		"listen_address": "0.0.0.0:7053"
	}
```

### 交易执行的基本过程
1. 根据意愿组装交易对象`Transaction`
2. 交易对象序列化(protocolbuffer3格式)为字节流 `transaction_blob`
3. 用私钥`skey`对`transaction_blob`签名得到`sign_data`，`skey`的公钥为`pkey`
4. 提交交易，见[提交交易](#提交交易)
5. 查询以确定交易是否成功或接收推送（websocket API）判断交易是否成功

### 试一试
如果您的区块链刚刚部署完成，那么目前区块链系统中只有创世账号。您可以通过http接口查询创世账号
`HTTP GET host:29333/getGenesisAccount`
您会得到类似这样的返回内容
```json
{
   "error_code" : 0,
   "result" :
   {
      "address" : "a00254d4b61e2169f26c6f1614202f978e800c78e5e651",
      "assets" : null,
      "balance" : 100000000000000000,
      "metadatas" : null,
      "priv" : {
         "master_weight" : 1,
         "thresholds" : {
            "tx_threshold" : 1
         }
      }
   }
}
```
返回结果中的`address`的值就是创世账号。
您还可以通过[查询账号](#查询账号)接口查询任意账号
```http
HTTP GET host:29333/getAccount?address=a00254d4b61e2169f26c6f1614202f978e800c78e5e651
```


## HTTP接口
### 查询账号

```http
GET /getAccount?address=a002423c235a7ba9649347ff85b6be1c51980d1eff0398&key=hello&code=xxx&issuer=xxx
```

功能:返回指定账号的信息及其所有资产、metadata

|参数|描述
|:--- | --- 
| address | 账号地址， 必填
| key | 账号的 metadata 中指定的key的值，如果不填写，那么返回结果中含有所有的metadata
| code, issuer |资产代码,资产发行商。这两个变量要么同时填写，要么同时不填写。若不填写，返回的结果中包含所有的资产。若填写，返回的结果中只显示由code和issuer指定的资产。
返回内容
```json
{
  "error_code" : 0,
  "result" : {
    "address" : "a002423c235a7ba9649347ff85b6be1c51980d1eff0398",
    "assets" : [//该账号的所有资产
      {
        "amount" : 1400,
        "property" : 
        {
          "code" : "CNY",
          "issuer" : "a002423c235a7ba9649347ff85b6be1c51980d1eff0398"
        }
      }, {
        "amount" : 1000,
        "property" : 
        {
          "code" : "USD",
          "issuer" : "a002423c235a7ba9649347ff85b6be1c51980d1eff0398"
        }
      }
    ],
    "assets_hash" : "9696b03e4c3169380882e0217a986717adfc5877b495068152e6aa25370ecf4a",
    "contract" : null,
    "metadatas" : [//该账号的所有metadata
      {
        "key" : "123",
        "value" : "123_value",
        "version" : 1
      }, {
        "key" : "456",
        "value" : "456_value",
        "version" : 1
      }, {
        "key" : "abcd",
        "value" : "abcd_value",
        "version" : 1
      }
    ],
    "nonce" : 1, //账号当前作为交易源执行过的交易数量。若nonce为0，该字段不显示
    "priv" : {
      "master_weight" : "1",
      "thresholds" : {
        "tx_threshold" : "1"
      }
    },
    "storage_hash" : "82c8407cc7cd77897be3100c47ed9d43ec4097ee1c00e2c13447187e5b1ac66c"
  }
}


```
- 如果该账号不存在,则返回内容

```json
{
   "error_code" : 4,
   "result" : null
}
```



### 查询交易

```http
GET /getTransactionHistory?hash=ad545bfc26c440e324076fbbe1d8affbd8a2277858dc35927d425d0fe644e698&ledger_seq=2
```

|参数|描述
|:--- | --- 
|hash | 用交易的唯一标识hash查询
|ledger_seq | 查询指定区块中的所有交易
上述两个参数产生的约束条件是逻辑与的关系，如果您同时指定两个参数，系统将在指定的区块中查询指定的交易

返回示例
```json
{
  "error_code": 0,
  "result":
  {
    "total_count": 1,
    "transactions": [
      {
        "close_time": 1506309263293181,
        "error_code": 0,
        "hash": "befe04be90d46fa8963d2fc2792d96e875995dc60734d6a6eb70392e49316ec0",
        "ledger_seq": 2,
        "signatures": [
          {
            "public_key": "b00204f82f1cc166e933b9829b4d6780ac2905f0f174bffc8e64bee5139f3dc9b3efe35041b89047152d468f26b6f490daf61244a101e0c78463f7b97153c7a8dbd3aaa9",
            "sign_data": "f7e87e461bc0a0ebba06aa236926f5fc6a3812a0d5d0f3a0e2268b32dd60dae57df339dbfbbea302e95a39cca6291bb80b59fe58788f33f99e00b1b3c12487ff"
          },
          {
            "public_key": "b002042ad76c8d526e6cb1bc2cd28214fd433eb1255018c1437f0bf2b081041309d6d5bb21c96292069f6ce760dab50883ca6b3fc478c9bb5ef6256717f6b31dd64eb2fe",
            "sign_data": "5478176325912dd6861748211f6ff51d6e49ab5c394a79112f6c656729637e33e6e9f14529d782ae3099f342ccfd3fc498d098e6f1c50a2a251262ffc1ab30aa"
          },
          {
            "public_key": "b00204e6e93bf0c8e0b2e0099cd54a677bcbeefa888c6bad992f55c05ba7f8169286963ae6968fc48c3c6b6a65827f2e9d604176a17919dfc665e8ea78a461c518757bb9",
            "sign_data": "7cabec78bdb8dbbd1e6fd7ac4c790280f59248a459e216c6902c444fa582008c770145225e00c9ac6b8d53c9851ac6fc7665af0a05f9e61d77597a2e80803381"
          }
        ],
        "transaction":
        {
          "nonce": 1,
          "operations": [
            {
              "create_account":
              {
                "dest_address": "a002fc445b8cec48783c7cdb49f69c67dc75893d053362",
                "metadatas": [
                  {
                    "key": "role",
                    "value": "我是货主"
                  }
                ],
                "priv":
                {
                  "master_weight": 10,
                  "thresholds":
                  {
                    "tx_threshold": 7
                  }
                }
              },
              "type": 1
            },
            {
              "create_account":
              {
                "dest_address": "a0025d8de2df8fa30a112f05c30ba4ecbd83975843d260",
                "metadatas": [
                  {
                    "key": "role",
                    "value": "我是操作员"
                  }
                ],
                "priv":
                {
                  "master_weight": 10,
                  "thresholds":
                  {
                    "tx_threshold": 7
                  }
                }
              },
              "type": 1
            }
          ],
          "source_address": "a00254d4b61e2169f26c6f1614202f978e800c78e5e651"
        }
      }
    ]
  }
}
```

如果没有查到交易则返回

```json
{
  "error_code": 4,
  "result":
  {
    "total_count": 0,
    "transactions": []
  }
}
```

###  查询区块头
```http
GET /getLedger?seq=xxxx&with_validator=true&with_consvalue=true
```
|参数|描述
|:--- | --- 
|seq  | ledger的序号， 如果不填写，返回当前ledger
|with_validator | true or false，是否显示验证节点列表
|with_consvalue | true or fasse，是否显示共识值

- 如果查询到ledger则返回内容:

```json
{
   "error_code" : 0,
   "result" : {
      "consensus_value" : {//with_consvalue为true时才出现
         "close_time" : 1506309624837046,
         "ledger_seq" : 12,
         "previous_ledger_hash" : "ff7670d9e201bef7f0c0ab06e63d643d2eb908d7583c1c873d189fdb5afd5352",
         "previous_proof" : "0afc010a2a080110022a24100b2220335c1fecca46814f86fe1fcc783910dbe5bd8997e3d0960128e858ff9d64868912cd010a8801623030323034316136633636343233373139373465613061663862366636393136353230343964366262396539653363663739383363363763613362336563616538363337346565306465616665666563393432383138303936613335313335343635373861346632643761623664353133396339323737656264306634633931623563643739641240c6c419163607620c248912928693e606170ba151b79795d3e3e932884df7cd4e91f1dc17c33f616db0d51354b10628d27573cb8bdd4128b3e5c71fc50b75e584",
         "previous_proof_plain" : {
            "commits" : [
               {
                  "pbft" : {
                     "commit" : {
                        "sequence" : 11,
                        "value_digest" : "335c1fecca46814f86fe1fcc783910dbe5bd8997e3d0960128e858ff9d648689"
                     },
                     "round_number" : 1,
                     "type" : 2
                  },
                  "signature" : {
                     "public_key" : "b002041a6c6642371974ea0af8b6f691652049d6bb9e9e3cf7983c67ca3b3ecae86374ee0deafefec942818096a3513546578a4f2d7ab6d5139c9277ebd0f4c91b5cd79d",
                     "sign_data" : "c6c419163607620c248912928693e606170ba151b79795d3e3e932884df7cd4e91f1dc17c33f616db0d51354b10628d27573cb8bdd4128b3e5c71fc50b75e584"
                  }
               }
            ]
         },
         "txset" : null
      },
      "header" : {
         "account_tree_hash" : "4fa53174df6c4d1d0410cbd1ee4c60d99ee884cc1e095bee4b633f1bada240d5",//账号树的根hash
         "close_time" : 1506309624837046,//区块生成时间
         "consensus_value_hash" ://共识值的hash "20a83a0522374368090e0c46d64f081c3404ac71975f288a2e7a90a3b01b302d",
         "hash" : "f8edfa81c9b45bb47fa49c90f784558b78ad4a243367c6cb8c6972dac4a0f9b7",//本区块hash
         "previous_hash" ://上个区块hash "ff7670d9e201bef7f0c0ab06e63d643d2eb908d7583c1c873d189fdb5afd5352",
         "seq" : 12,//区块的序号
         "tx_count" : 1,//截至到本区块，已经产生的所有交易个数
         "validators_hash" ://共识节点集合hash "b744ccae0ecd1b0c09923564bc1556949e96f78755bdbcec06a82ddad420cca4",
         "version" : 3000//区块版本号
      },
      "validators" : [//with_validator为true时才出现
          "a002953b57b025662c9f9de93df4c319cc39bec181c2b5" 
      ]
   }
}
```

- 如果没有查询到ledger返回的内容:

``` json
{
   "error_code" : 4,
   "result" : null
}
```

### 提交交易

```http
POST /submitTransaction
```
数据格式
```json
{
  "items" : [{
      "transaction_blob" : "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
      "signatures" : [{
          "sign_data" : "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
          "public_key" : "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        }, {
          "sign_data" : "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
          "public_key" : "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        }
      ]
    }
  ]
}
```
|参数|描述
|:--- | --- 
|transaction_blob| 交易序列化之后的16进制格式。您可以参照[定义交易](#定义交易)来组装自己的交易数据
|sign_data| 签名数据， 16进制格式。其值为对transaction_blob进行签名(动词)得到的签名数据。
|public_key| 公钥， 16进制格式。

## 定义交易
相关的protocolbuffer的数据格式
```json
//交易
message Transaction
{
    enum Limit{
            UNKNOWN = 0;
            OPERATIONS = 1000;
    };
    string source_address = 1;
    int64 nonce = 2;
    string expr_condition = 3;
    repeated Operation operations = 4;
    bytes metadata = 5;
    int64  fee = 6;
}

//操作父类
message Operation
{
    enum Type {
        UNKNOWN = 0;
        CREATE_ACCOUNT = 1;
        ISSUE_ASSET = 2;
        PAYMENT = 3;
        SET_METADATA = 4;
        SET_SIGNER_WEIGHT = 5;
        SET_THRESHOLD = 6;
        PAY_COIN = 7;
    };
    Type type = 1;
    string source_address = 2;
    bytes metadata	= 3;
    string expr_condition = 4;

    OperationCreateAccount create_account = 5;
    OperationIssueAsset issue_asset = 6;
    OperationPayment payment = 7;
    OperationSetMetadata set_metadata = 9;
    OperationSetSignerWeight set_signer_weight = 10;
    OperationSetThreshold set_threshold = 11;
    OperationPayCoin pay_coin = 12;
}
```

交易Transaction有5个关键字段
- source_address: 交易源账号，即交易发起方的账号。当这笔交易成功后，交易源账号的nonce字段会自动加1。账号中的nonce意义是本账号作为交易源执行过的交易数量。

- nonce：其值必须等于交易源账号的当前nonce+1，这是为了防止重放攻击而设计的。如何查询一个账号的nonce可参考[查询账号](#查询账号)。若查询账号没有显示nonce值，说明账号的当前nonce是0.

- expr_condition：针对本交易的表达式，高级功能。详见[表达式](#表达式)

- operations：操作列表。本交易的有效负载，即本交易想要做什么事情。见[操作](#操作)

- metadata：用户自定义字段，可以不填写，备注用。

Operation是所有操作的父类：
- type: 操作类型的枚举。如其值为ISSUE_ASSET（发行资产），那么本操作中的issue_asset字段就会被使用；如果其值为PAYMENT，那么本操作中payment字段就会被使用……详见[操作码](#操作码)

- source_address：操作源，即本操作针对哪个账号生效。若不填写，则默认本操作源等于本操作源。
- metadata：本操作的备注，用户自定义，可以不填写
- expr_condition：针对本操作的表达式，若您用不着这个功能，可以不填写。详见[表达式](#表达式)

### 操作码
|代码 | 枚举名 | 说明
|:--- | --- | --- |
|1  | CREATE_ACCOUNT | 创建账号
|2  | ISSUE_ASSET | 发行资产
|3  | PAYMENT | 转移资产
|4  | SET_METADATA | 设置metadata
|5  | SET_SIGNER_WEIGHT | 设置权重
|6  | SET_THRESHOLD | 设置门限

### 操作

#### 创建账号
- 功能
  在区块链上创建一个新的账号
- 成功条件
  - 各项参数合法
  - 要创建的账号不存在

- protocol结构定义
    ```json
    message OperationCreateAccount
    {
        string dest_address = 1;
        Contract contract = 2;
        AccountPrivilege priv = 3;
        repeated KeyPair metadatas = 4;	
        int64	init_balance = 5;
    }
    ```

    - dest_address：要创建的账号的地址
    - contract:合约。若你想要创建一个不具有合约功能的账号，可以不填写这部分。若您想创建具有合约功能的账号，请参照[合约](#合约)
    - priv：账号的初始权力分配。相关的数据结构定义：
        ```json
        message OperationTypeThreshold
        {
            Operation.Type type = 1;
            int64 threshold = 2;
        }

        message AccountPrivilege 
        {
            int64 master_weight = 1;
            repeated Signer signers = 2;
            AccountThreshold thresholds = 3;
        }
        message Signer
        {
            enum Limit{
                SIGNER_NONE = 0;
                SIGNER = 100;
            };
            string address = 1;
            int64 weight = 2;
        }
        message AccountThreshold
        {
            int64 tx_threshold = 1; //required, [-1,MAX(INT64)] -1: 表示不设置
            repeated OperationTypeThreshold type_thresholds = 2; //如果这个设置，则操作门限以这个为准
        }
        ```
    若你想创建一个不受其他账号控制的账号。将priv.master_weight设置为1，将priv.thresholds.tx_threshold设为1即可。若您想创建一个受其他账号控制的账号，参见[控制权的分配](#控制权的分配)
    - metadatas：metadata列表。您可以为新建的账号设置一批初始的metadata。其数据类型为KeyPair,结构如下
    ```json
    message KeyPair
    {
        string key = 1;
        string value = 2;
        int64 version = 3;
    }
    ```
    这是一个版本化的键值对数据库，如果您不需要，可以不填写这部分。
    - init_balance:暂时未启用


#### 发行资产
- 功能
  操作源账号发行一笔数字资产，执行成功后操作源账号的资产余额中会出现这一笔资产
- 成功条件
  - 各项参数合法
  - 资产数量没有溢出（int64）

- protocol结构定义
    ```json
    message OperationIssueAsset
    {
        string code = 1;
        int64 amount = 2;
    }
    ```
    - code:要发行的资产代码，长度范围[1, 64]
    - 发行的数量。数值范围(0,MAX(int64))
成功条件
执行成功后，操作源的资产表中会出现这部分新发行的资产

#### 设置metadata
- 功能
  操作源账号修改或添加一个metadata到自己的metadata表中
- 成功条件
  - 各项参数合法

- protocol结构定义
    ```json
    message OperationSetMetadata
    {
        string key = 1;
        string value = 2;
        int64 version = 3;
    }
    ```
    - key: 主键，账号内唯一。长度范围[1,1024]
    - value: 值。长度范围[0,1M]
    - version: 版本号，可以不填写。若您想使用这个高级功能，参见[版本化控制](#版本化控制)

#### 转移资产
- 功能
  操作源账号将一笔资产转给目标账号
- 成功条件
  - 各项参数合法
  - 源账号该类型的资产数量足够

- protocol结构定义
    ```json
    message OperationPayment
    {
        string dest_address = 1;

        Asset asset = 2;

        string input = 3;
    }
    ```
    - dest_address: 资产接收方账号地址
    - asset: 要转移的资产
    ```json
    message Asset
    {
         AssetProperty property = 1; //资产属性
         int64 amount = 2; //数量
    }

    message AssetProperty
    {
         string issuer = 1; //资产发行方
         string code = 2; //资产代码
    }
    ```
    - input: 本次转移触发接收方的合约，合约的执行入参就是input



#### 设置权重
- 功能
  设置签名者拥有的权重
- 成功条件
  - 各项参数合法

- protocol结构定义
    ```json
    message OperationSetSignerWeight
    {
         int64 master_weight = 1; //required, [-1,MAX(UINT32)] -1: 表示不设置
         repeated Signer signers = 2; //address:weight, 如果weight 为0 表示删除这个signer
    }
    ```
    - master_weight：本账号地址拥有的权力值
    - 各个签名者的权力值, Singer的定义如下
    ```json
    message Signer 
    {
    enum Limit{
            SIGNER_NONE = 0;
            SIGNER = 100;
    };
         string address = 1;
         int64 weight = 2;
    }
```

#### 设置门限
- 功能
  设置各个操作所需要的门限
- 成功条件
  - 各项参数合法

- protocol结构定义
```json
message OperationSetThreshold
{
	    int64 tx_threshold = 1;
	    repeated OperationTypeThreshold type_thresholds = 4; //type:threshold ，threshold:0 表示删除这个类型的type
}
```
OperationTypeThreshold的结构定义如下
```json
message OperationTypeThreshold{
	 Operation.Type type = 1;  //操作码，哪种操作
	 int64 threshold = 2;    //代表这种操作所需的权重门限
}
```


## 高级功能

### 控制权的分配
您在创建一个账号时，可以指定这个账号的控制权分配。您可以通过设置priv的值设置。下面是一个简单的例子

```json
{
    "master_weight": 70,//本地址私钥拥有的权力值 70
    "signers": [//分配出去的权力
        {
            "address": "a0023db6c3e25fb110ae596a57ebbb69900b6fda651534",
            "weight": 55	//上面这个地址拥有权力值55
        },
        {
            "address": "a002567bcf969137a7b4f910dbbde35a0de8450c641e08",
            "weight": 100	//上面这个地址拥有权力值100
        }
    ],
    "thresholds"://不同的操作所需的权力阈值
    {
        "tx_threshold": 8,//发起交易需要权力值 8
        "type_thresholds": [
            {
                "type": 1,//创建账号需要权利值 11
                "threshold": 11
            },
            {//发行资产需要权利值 21
                "type": 2,
                "threshold": 21
            },
            {//转移资产需要权力值 33
                "type": 3,
                "threshold": 31
            },
            {//设置metadata需要权利值 41
                "type": 4,
                "threshold": 41
            },
            {//变更控制人的权力值需要权利值 51
                "type": 5,
                "threshold": 51
            },
            {//变更各种操作的阈值需要权利值 61
                "type": 6,
                "threshold": 61
            }
        ]
    }
}
```

### 版本化控制
每一个账号的metadata都是一个版本化的小型数据库。版本化的特点是可以避免修改冲突问题。


### 表达式
该表达式字段，用于自定义交易有效规则，比如设置交易在某个账户的master_weight 大于 100 有效，则填：

```JavaScript
jsonpath(account("bubiV8i6mtcDN5a1X7PbRPuaZuo63QRrHxHGr98s"), ".priv.master_weight") > 100
```

表达式可用的函数

|参数|描述
|:--- | --- |
|account(address)| 获取账户信息, 返回 json 序列化字符串
|jsonpath(json_string, path)| 获取json对象的属性值 
|LEDGER_SEQ| 内置变量，代表最新的区块高度
|LEDGER_TIME|内置变量，代表最新的区块生成时间
|`( )` `=`  | 嵌套括号，函数调用如sqrt(x)
|`*` `/`  | 乘法、除法
|`+` `-`      | 加法、减法
|`<` `<=`  `>`  `>=`  `==`  `!=` | 比较运算
| `&&`  |  与 运算
| &#124; |或运算
|,|  逗号运算

>例：Alice发起了一笔交易，她想要使这笔交易在5秒内有效。她发现当前时间戳(微秒)是 `1506393720000000`，5秒之后也就是 `1506393725000000`。那么她就可以在交易中的`expr_condition`字段写上下面这句
```json
LEDGER_TIME >= 1506393720000000 && LEDGER_TIME <= 1506393725000000
```
这句话的意思是，该交易只在时间范围`[1506393720000000, 1506393725000000]`内有效。
过段时间之后，Alice发现当前时间已经超过1506393725000000，那么Alice就可以断定这笔交易要么已经被处理，要么已经彻底失效。

### 合约
合约是一段JavaScript代码,标准(ECMAScript as specified in ECMA-262)。合约的入口函数是main函数，您写的合约代码中必须有main函数的定义。该函数的入参是字符串input，是调用该合约的时候指定的。
下面是一个简单的例子
```javascript
function foo(bar)
{
  /*do whatever you want*/

}
function main(input)
{
  var para = JSON.parse(input);
  if (para.do_foo)
  {
    var x = {
      'hello' : 'world'
    };
    foo(x);
  }
}
```

系统提供了几个全局函数, 这些函数可以获取区块链的一些信息，也可驱动账号发起交易
== 注意，自定义的函数和变量不要与内置变量和全局函数重名，否则会造成不可控的数据错误。 ==
#### 内置函数

- ##### 获取账号信息(不包含metada和资产)

	`callBackGetAccountInfo(address);`
	 address: 账号地址

    例如
    ``` javascript
    var account = callBackGetAccountInfo('a0025e6de5a793da4b5b00715b7774916c06e9a72b7c18');
    /*
    account具有如下格式
     {
        "address": "a002943cede1be5fb0ca0da9f9b49b0ce20b613357524a",
        "assets_hash": "5aef61a8988ce2be1da67cf4b37717748c352b8e4a0bdad2ad0964f80aca0101",
        "contract": null,
        "priv": null,
        "storage_hash": "e4775fb7fc2a5a06a4bbe0e63f362f8e24ff7752f0259ccd2fe1fc2e6e68781a"
      }
    */
    ```

- ##### 获取某个账号的metadata信息
    `callBackGetAccountMetaData(account_address, metadata_key);`
    - account_address: 账号地址
    - metadata_key： metadata的key

    例如
    ```javascript
    var bar = callBackGetAccountMetaData('a0025e6de5a793da4b5b00715b7774916c06e9a72b7c18','abc');
    /*
     bar的值是如下的格式
     {
         'key':'abc',
         'value':'hello world',
         'version':12
     }
    */

    ```
    即可得到账号a0025e6de5a793da4b5b00715b7774916c06e9a72b7c18的metadata中abc的值

- ##### 获取某个账号的资产信息

    `callBackGetAccountAsset(account_address, asset_property);`

    - account_address: 账号地址
    - asset_property： 资产属性

    例如
    ```javascript
    var asset_property =
    {
      'issuer' : 'a002bbe0b6f547d6bec2c83fb9bb93e75d37c1755f2de6',
      'code' : 'CNY'
    };
    var bar = callBackGetAccountAsset('a0025e6de5a793da4b5b00715b7774916c06e9a72b7c18', asset_property);

    /*
    {
      "amount": 1,
      "property": {
        "code": "CNY",
        "issuer": "a002bbe0b6f547d6bec2c83fb9bb93e75d37c1755f2de6"
      }
    }
    */
    ```

- ##### 获取区块信息

    `callBackGetLedgerInfo(ledger_seq);`
    - ledger_seq: 区块号

    例如
    ```javascript
    var ledger = callBackGetLedgerInfo(40);
    /*
    ledger具有如下格式
    {
        "account_tree_hash": "af05a60772cfd39f3b7838f4032f50450c100dedddf88e0132066688f6ae5c14",
        "consensus_value": {
          "close_time": 1495855656157405,
          "payload": "240398d89a5efba398fefb0dc194b45abe7b9dbc35326ee8238fff6633371004"
        },
        "hash": "9f82d8ad1c381e1ce2ce00c559fb2cf3a386d79e9414e92ce3ed809258913384",
        "ledger_sequence": 40,
        "ledger_version": 1000,
        "previous_hash": "3ff9b79479d62e7c52f2c0ab08598d219ffd4403bd5c1337764d3591e9b0ba24",
        "transaction_tree_hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
        "tx_count": 34359738368
      }
    */

    ```


- #####  做交易
    令合约账号做一笔交易，即里面的任意一个操作的source_address都会自动变成合约账号。
    所以source_address是不需要填写的，即使填写也无用。

    `callBackDoOperation(transaction);`
    - transaction: 交易内容
    返回: true/false
    
    例如
    ```javascript
    var transaction =
    {
      'operations' :
      [
        {
          "type" : 2,
          "issue_asset" :
          {
            "amount" : 1000,
            "code" : "CNY"
          }
        }
      ]
    };

    var result = callBackDoOperation(transaction);
    ```

#### 内置变量
- #####  该合约账号的地址
    thisAddress

    全局变量```thisAddress```的值等于该合约账号的地址。

    例如账号x发起了一笔交易调用合约Y，本次执行过程中，thisAddress的值就是Y合约账号的地址。

    ```javascript
    var bar = thisAddress;
    /*
    bar的值是Y合约的账号地址。
    */
    ```

- ##### 调用者的地址
    sender
    ```sender``` 的值等于本次调用该合约的账号。

    例如某账号发起了一笔交易，该交易中有个操作是调用合约Y（该操作的source_address是x），那么合约Y执行过程中，sender的值就是x账号的地址。

    ```javascript
    var bar = sender;
    /*
    那么bar的值是x的账号地址。
    */
    ```

- ##### 触发本次合约的交易
    trigger

    ```trigger``` 的值等于触发本次合约的交易。

    例如某账号A发起了一笔交易tx0，tx0中有一个操作是给某个合约账户转移资产(调用合约), 那么```trigger```的值就是交易tx0。

    ```javascript
    var bar = trigger;
    /*
    那么bar的值是触发本次合约的交易。
    */
    ```

- ##### 触发本次合约调用的操作的序号
    triggerIndex

    ```triggerIndex``` 的值等于触发本次合约的操作的序号。

    例如某账号A发起了一笔交易tx0，tx0中第0（从0开始计数）个操作是给某个合约账户转移资产(调用合约), 那么```triggerIndex```的值就是0。

    ```javascript
    var bar = triggerIndex;
    /* bar 是一个非负整数*/
    ```

- ##### 本次共识数据
    consensusValue

    ```consensusValue``` 当前块(正在生成的块)的共识数据。
    consensusValue的数据结构可以在src/proto/chain.proto中找到ConsensusValue。
    ConsensusValue是一个protobuffer对象，根据protobuffer对象转为json格式的标准方法转换，即是```consensusValue```的值。

    ```javascript
    var bar = consensusValue;
    /*consensusValue结构比较复杂，常用的数据有以下几个:*/
    consensusValue.close_time;	/*当前时间,也就是区块生成时间*/
    consensusValue.ledger_seq; 	/*当前区块序号*/
    consensusValue.previous_ledger_hash; /*上一个区块hash*/
    ```

## 错误码
错误由两部分构成:
- error_code : 错误码，大概的错误分类
- error_desc : 错误描述，一般能从错误描述准确发现错误具体信息

错误列表如下：

|error_code | enum | error_desc
|:--- | --- | --- |
|0  | ERRCODE_SUCCESS | 操作成功
|1  | ERRCODE_INTERNAL_ERROR | 服务内部错误
|2  | ERRCODE_INVALID_PARAMETER | 参数错误
|3  | ERRCODE_ALREADY_EXIST | 对象已存在， 如重复提交交易
|4  | ERRCODE_NOT_EXIST | 对象不存在，如查询不到账号、TX、区块等
|5  | ERRCODE_TX_TIMEOUT | TX 超时，指该 TX 已经被当前节点从 TX 缓存队列去掉，==但并不代表这个一定不能被执行==
|20 | ERRCODE_EXPR_CONDITION_RESULT_FALSE | 指表达式执行结果为 false，意味着该 TX 当前没有执行成功，==但这并不代表在以后的区块不能成功==
|21 | ERRCODE_EXPR_CONDITION_SYNTAX_ERROR | 指表达式语法分析错误，代表该 TX 一定会失败
|90 | ERRCODE_INVALID_PUBKEY | 公钥非法 
|91 | ERRCODE_INVALID_PRIKEY | 私钥非法
|92 | ERRCODE_ASSET_INVALID | 资产issue 地址非法|code长度不在有效范围内|数额不在有效范围
|93 | ERRCODE_INVALID_SIGNATURE | 签名权重不够，达不到操作的门限值
|94 | ERRCODE_INVALID_ADDRESS | 地址非法
|97 | ERRCODE_MISSING_OPERATIONS | 交易缺失操作
|99 | ERRCODE_BAD_SEQUENCE | 交易序号错误
|100| ERRCODE_ACCOUNT_LOW_RESERVE | 余额不足
|101| ERRCODE_ACCOUNT_SOURCEDEST_EQUAL | 源和目的账号相等
|102| ERRCODE_ACCOUNT_DEST_EXIST | 创建账号操作，目标账号已存在
|103| ERRCODE_ACCOUNT_NOT_EXIST | 账户不存在
|104| ERRCODE_ACCOUNT_ASSET_LOW_RESERVE | 支付操作，资产余额不足
|105| ERRCODE_ACCOUNT_ASSET_AMOUNT_TOO_LARGE |资产数量过大，超出了int64的范围
|114| ERRCODE_OUT_OF_TXCACHE |  TX 缓存队列已满
|120| ERRCODE_WEIGHT_NOT_VALID | 权重值不在有效范围内
|121| ERRCODE_THRESHOLD_NOT_VALID | 门限值不在有效范围内
|144| ERRCODE_INVALID_DATAVERSION | metadata的version版本号不与已有的匹配（一个版本化的数据库）
|151| ERRCODE_CONTRACT_EXECUTE_FAIL | 合约执行失败
|152| ERRCODE_CONTRACT_SYNTAX_ERROR | 合约语法分析失败







