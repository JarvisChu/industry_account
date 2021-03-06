[TOC]

# 1 微信授权场景

## 1.0 小程序登录

请求方式POST

```
https://{CloudIndustryHost}/qywx/wx/login
```

请求包体

```json
{
    "WxCode": "code"
}
```

| 参数名称 | 必选  | 描述 |
| --- | --- | --- |
| WxCode | 是 | wx.login获取到的code, 详见[小程序登录](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html) |

返回Header

```
Set-Cookie: sessionid=wedfapehfawefjae;
```

> SessionId是会话id，在后端关联了当前会话的openid和session_key。
> 小程序应持久化存储，在后续每次wx.request()时带上。详细说明见[小程序登录](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html)

## 1.1a 请求用户微信授权

详见[小程序授权](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/authorize.html)

小程序请求授权后，可以拿到用户的nickname，在1.3里一起上报给工业云，换取工业云access token

## 1.2a 查询用户信息

详见[小程序获取用户信息](https://developers.weixin.qq.com/miniprogram/dev/api/open-api/user-info/wx.getUserInfo.html)

## 1.3 绑定手机、昵称，换取工业云登录态

> 对防疫场景，仅需求方（医院侧）需要

请求方式POST

```
https://{CloudIndustryHost}/qywx/wx/bind_user
```

请求Header

```
Cookie: sessionid=asfwefbqwbfwqef;
```

请求包体

```json
{
    "EncrptedMobile": "asdfbwuefaoefn",
    "NickName": "张三",
    "Iv": "asdfaefaef"
}
```

| 参数名称 | 必选  | 描述 |
| --- | --- | --- |
| EncrptedMobile | 是 | 加密手机号 |
| NickName | 是 | 昵称 |
| Iv | 是 | 加密初始向量 |

返回包体

```json
{
    "Code": 0,
    "Msg": "",
    "UserId": "user id",
    "CorpId": "corp id",
    "AccessToken": "access token"
}
```

| 参数 | 必选 | 含义 |
| ---- | --- | --- |
| Code | 是 | 0: 成功, 非0: 失败 |
|  UserId | 是 | 手机号对应的工业云user id|
|  AccessToken | 是 | 手机号对应的工业云access token|

> 异常情况：该手机号未在工业云注册过，则需前端提示医院侧前往sidacloud.com进行注册、需求发布

## 1.4 换取会话中的工业云登录态

请求方式GET

```
http://{CloudIndustryHost}/qywx/wx/token
```

请求Header

```
Cookie: sessionid=asfaefaef;
```

返回包体

```json
{
    "Code": 0,
    "AccessToken": "token",
    "UserId": "userid"
}
```

# 2. 我要供货场景

## 2.1 读企业信息

> 供货方调用

请求方式POST

```
http://{CloudIndustryHost}/iam/api/v1/corps?access_token=ACCESS_TOKEN
```



请求JSON包体

```
{
    "CorpIds": ["corp id"]
}
```

| 参数名称 | 必选  | 描述 |
| --- | --- | --- |
| CorpIds | 是   | 企业的id列表，最多不能超过50个|

返回JSON包体

```
{
   "Code" : 0,
   "Msg" : "ok",
   "Corps": [
        {
            "CorpId": "id",
            "Name": "name",
            "Tel": "tel",
            "Contact": "contact",
        }
   ]
}
```

| 参数名称 | 必选  | 描述 |
| --- | --- | --- |
| Name | 否 | 企业名称 |
| Tel | 否 | 联系电话 |
| Contact | 否 | 联系人姓名 |

## 2.2 供货（我要供货)

请求方式POST

```
https://{CloudIndustryHost}/asm/api/supply?access_token={access token}
```

请求包体

```json
{
    "CorpName": "企业名称",
    "Contact": "联系人名称",
    "Tel": "13700000000",
    "DemandId": "xxx", //需求id
    "CorpId":"123", //投标商企业id
    "Price":2005, //服务报价（单位分）
    "Count": 200000 //数量
}
```

| 参数名 | 是否必填 | 说明 |
| ------ | ------ | ------ |
| CorpName | 是 | 企业名称 |
| Contact | 是 | 联系人姓名 |
| Tel | 是  | 联系电话 |
| DemandId | 是 | 需求id |
| CorpId | 是 | 投标商企业id，无企业时填"0"，不能为空 |
| Price | 是 | 服务报价（单位分）|
| Count | 是 | 数量 |

返回包体

```json
{
    "Code": 0,
    "Msg": ""
}
```


## 2.4 读投标列表

请求方式POST

```
https://{CloudIndustryHost}/asm/api/bid_list?access_token={access_token}
```

请求包体

```json
{
    "Filters": [
        {
            "Name": "demand_id",//需求id字段
            "Op": "in", //需求
            "Values": [
                "12341",
                "12342",
                "12344"
            ]
        }
    ],
    "Sorts": [//排序
        {
            "Name": "demand_rating",//排序字段
            "OrderBy": "desc"//默认asc生序，desc降序
        }
    ],
    "PageSize": 10,
    "Offset": 4
}
```

取个人端/企业端消息

根据个人/企业的企业id获取供货的列表

```json
{
    "Filters": [
        {
            "Name": "service_corp_id",
            "Op": "==",
            "Values": ["4332237717172222"] //个人/企业的企业id，从用户信息接口处获取
        },
        {
            "name": "demand_type",
            "op": "==",
            "values": [
                "realobject"
            ]
        }
    ],
    "Sorts": [
        {
            "Name": "created_at", //根据供货创建时间倒序排列
            "OrderBy": "desc"
        }
    ],
    "PageSize": 10,
    "Offset": 0
}
```

取医院端企业需求

先通过`3.3 查询已发布的需求(列表)详情`获取医院发布的所有实物需求列表（需求选择所需），选择一个需求获取需求的已投标列表（参数见下）

```json
{
    "Filters": [
        {
            "Name": "demand_id",
            "Op": "==",
            "Values": ["130"]  //需求id
        },
        {
            "name": "demand_type",
            "op": "==",
            "values": [
                "realobject"
            ]
        }
    ],
    "Sorts": [
        {
            "Name": "created_at", //根据供货创建时间倒序排列
            "OrderBy": "desc"
        }
    ],
    "PageSize": 10,
    "Offset": 0
}
```

| 参数名 | 是否必填 | 说明 | 类型 |
| ------ | ------ | ------ | --- |
| Filters | 否 | 过滤条件(如果是sp)，可以为空，如果是cp不能为空 | object array |
| Sorts | 否 | 排序条件 | object array |
| PageSize | 否 | 分页查找一次获取的数量 | int |
| Offset | 否 | 分页查找起始位置 | int |

过滤条件Sorts说明

| 参数名 | 是否必填 | 说明 |
| ------ | ------ | ------ |
| Name | 是 | 字段名 |
| OrderBy | 是 | 排序方式支持降序和升序："desc"、"asc" |

过滤条件Filters说明

| 参数名 | 是否必填 | 说明 |
| ------ | ------ | ------ |
| Name | 是 | 字段名 |
| Op | 是 | 比较方式 |
| Values | 是 | 取值列表，字符串数组 |

投标字段名Name说明
	
| 取值 |  说明 |
| ------ |------ |
| created_at | 投标信息创建时间 | 
| bid_id | 创建该需求的企业ID | 
| bid_status | 投标状态 |
| demand_id | 需求id |
| demand_rating | 需求评分 |
| demand_comment | 需求评价信息 |
| service_corp_id | 投标商企业id |
| service_price | 服务报价（单位元）|
| service_period | 服务周期（单位月）|
| service_description | 服务描述 |
| service_doc | 服务相关文档 | 
| service_rating | 服务商评分 |
| service_comment | 服务商评价信息 |

比较方式说明：

| 取值 |  说明 |
| ------ |------ |
| > |  | 
| < |  | 
| >= |  | 
| <= |  | 
| == |  | 
| like | 数据库操作：类似于 | 
| in | 处于Values列表中的值 | 
| neq | 仅用于比较数字，不等于 | 
| != | 仅用于比较数字，不等于 | 


返回包体

```json
{
    "Code": 0,
    "Msg": "成功",
    "Total": 3,
    "Infos": [
        {
            "CreatedAt": "1550720866",
            "BidId": "15507208663542",
            "DemandId": "12341",
            "DemandName": "20万瓶消毒液",
            "DemandRating": 5,
            "ServiceCorpId": "133334",
            "ServiceCorpName": "xxx",
            "ServiceCorpLogo": "xxx", // 服务商logo
            "ServicePrice": 2330, //除100后是价格
            "ServicePeriod": 30,
            "ServicePeriodUint": 1,
            "ServiceDescription": "投标了",
            "ServiceDoc": "",
            "Rating": 4,
            "BidStatus": 0,
	        "Guarantee": {
	            "RealNameAuthStatus": 2,
	            "OfflineCertStatus": 0,
	            "EarnestMoneyStatus": 0
            },
            "ServicePhoneNum": "13512340987",
            "ServiceContactUser": "供货人",
            "ServiceContactUserId": "45223843256526161",
            "ServiceContactEmail": "xxx@126.com"
        },
    ]
}
```

| 参数名 | 基础字段 | 说明 | 类型 |
| ------ | ------ | ------ | --- |
| Code | 是 | 错误码 | int |
| Msg | 是 | 错误说明 | string |
| Infos | 否 | 需求列表 | array of object |
| Total | 是 | 符合条件的总数量，分页的时候可能用来判断是否有更多 | int |
| BidId | 是 | 投标id | string |
| DemandId | 是 | 需求id | string |
| ServiceCorpName | 是 | 投标商企业名 | string |
| ServicePrice | 是 | 投标单价，单位元 | float |


设计交互与字段对应关系

| 交互字段 | 对应字段 |
|:--- |:--- |
| 供货需求 | DemandName |
| 提交时间 | CreatedAt |
| 企业名称 | ServiceCorpName |
| 现货数量 | ServicePeriod |
| 现货单价 | ServicePrice |
| 联系人 | ServiceContactUser|
| 手机号码 | ServicePhoneNum |


# 3.需求撮合
> 需求撮合相关接口

## 3.1 获取分类列表

请求方式GET

```
https://{CloudIndustryHost}/asm/api/colla_category/:CategoryID?access_token={token}&Depth=2
```

| 参数名 | 是否必填 | 说明 |
| ------ | ------ | ------ |
| CategoryID | 是 | 分类ID，id=0，表示从根目录查找 |
| Depth | 是 | 要查找的子分类层数，depth=0表示只查询id对应的信息，必须为整数 |

返回包体

```json
{
    "Code": 0,
    "Msg": "成功",
    "CategoryTree": [{
        "CategoryID": "2003",
        "Name": "工业类",
        "Description": "分类备注",
        "ParentID": 20,
        "SubCategoryList":[
            {
                "CategoryID": "200301",
                "Name": "SaaS服务",
                "Description": "分类备注",
                "ParentID": 2003
            }
        ]
    }]
}
```

| 参数名 | 基础字段 | 说明 |
| ------ | ------ | ------ |
| Code | 是 | 错误码 |
| Msg | 是 | 错误说明 |
| CategoryTree | 否 | 查询到的分类树信息类 |
| CategoryID | 否 | 分类ID |
| Name | 是 | 分类名称，用于展示 |
| Description | 否 | 分类备注 |
| ParentID | 是 | 上一级分类，如果当前分类已经是最上一级，则=0 |
| Quantity | 否 | 分类下的产品数量 |
| SubCategory | 否 | 子分类信息 |

## 3.2 查询需求(列表)详情

请求方式GET

```
https://{CloudIndustryHost}/asm/api/colla_demands?access_token={token}
```

查询需求的时候有两种查询方式，需要带的参数都是不同的，如下表：

| 参数名 | 是否必填 | 说明 |
| ------ | ------ | ------ |
| DemandIDs | 否 | 需求ID列表，例如：[1,10] |
| CorpID | 否 | CP的ID，可带其他过滤条件 |
| CommonParams | 否 | 只有带CorpID的时候生效，过滤排序条件，将进行与操作 |

CommonParams结构如下：

```
type CommonQueryParams struct {
	Offset               uint32    
	Limit                uint32 
	Sorts                []*Sort
	Filters              []*Filter
}

type Sort struct {
	Name                 string 
	OrderBy              string 
}

type Filter struct {
	Name                 string 
	Op                   string  
	Values               []string
}

```

过滤条件Sort说明

| 参数名 | 是否必填 | 说明 |
| ------ | ------ | ------ |
| Name | 是 | 字段名 |
| OrderBy | 是 | 排序方式支持降序和升序："desc"、"asc" |

过滤条件Filter说明

| 参数名 | 是否必填 | 说明 |
| ------ | ------ | ------ |
| Name | 是 | 字段名 |
| Op | 是 | 比较方式 |
| Values | 是 | 取值列表，字符串数组 |

需求字段名Name说明（此处列出常用的，详细支持的取值参照前面的定义中注释的全小写字母部分）：
	
| 取值 |  说明 |
| ------ |------ |
| id | 需求ID | 
| corp_id | 创建该需求的企业ID | 
| name | 标题 | 
| category_id | 分类ID，只支持in 或者 == | 
| budget_type | 价格类型 | 
| budget_min | 价格最低值 | 
| budget_max | 价格最高值 | 
| description | 描述 | 
| bidding_completion_date | 预计完成时间 | 
| bidding_deadline | 竞标截止时间 | 
| anonymous | 是否匿名 | 
| state | 状态 | 

比较方式说明：

| 取值 |  说明 |
| ------ |------ |
| > |  | 
| < |  | 
| >= |  | 
| <= |  | 
| == |  | 
| like | 数据库操作：类似于 | 
| in | 处于Values列表中的值 | 
| neq | 仅用于比较数字，不等于 | 
| != | 仅用于比较数字，不等于 | 

**注意**：DemandIDs、CorpID两种参数必须带一个。

举例说明：

1. 查询指定ID为1和10的需求

		https://{CloudIndustryHost}/asm/api/colla_demands?DemandIDs=[1,10]
	
2. 查询企业ID为1创建的需求

		https://{CloudIndustryHost}/asm/api/colla_demands?CorpID=1
	
返回包体示例

```json
{
    "Code": 0,
    "Msg": "成功",
    "DemandList": [
        {
            "ID": 36,
            "Name": "火灾识别",
            "CategoryIDList": [3],
            "CategoryList": [
                {
                    "ID": 3,
                    "Name": "最大分类1asd",
                    "Description": "测试"
                }
            ],
            "Logo": "logourl",
            "Standard": "国药甲1",
            "Location": "杭州",
            "Description": "详细说明(富文本)",
            "HeadUserIDList": [1,2],
            "HeadUserList": [
            		{
            			"UserID":1,
            			"UserName":"张三",
            			"Email":"aaa@126.com",
            			"Mobile":"111222",
            		},
            		{
            			"UserID":2,
            			"UserName":"李四",
            			"Email":"aba@126.com",
            			"Mobile":"11122242",
            		},
            ],
            "Anonymous": false,
            "State": 2,
            "CorpID": 0,
            "CorpName": "",
            "CreateTime": 1550807412,
            "UpdateTime": 1550807412,
            "ViewsCount": 0,
            "DemandRating": 0,
            "BidCount": 0,
            "AppendixList": [
                {
                    "Name": "AAA",
                    "Url": "132.232.86.146:18081/10,01992a43273a"
                }
            ],
            "Bidding": {
                "RecvObject": "aa",
                "RecvLocation": "asa",
                "CompletionDate": 1648315396,
                "BiddingDeadline": 1648315396,
                "TimeNodeList": [
                    {
                        "Timestamp": 11,
                        "Description": "aaa"
                    }
                ]
            },
            "Budget": {
                "BudgetType": 1,
                "Min": 1,
                "Max": 2
            }
        }
    ],
    "Count": 1
}
```

| 参数名 | 基础字段 | 说明 | 类型 |
| ------ | ------ | ------ | --- |
| Code | 是 | 错误码 | int |
| Msg | 是 | 错误说明 | string |
| DemandList | 是 | 需求列表 | object array |
| Count | 是 | 符合条件的总数量，分页的时候可能用来判断是否有更多 | int |
| CategoryIDList | 是 | 需求所属分类id | int array |
| Logo | 否 | 需求logo | string |
| Standard | 否 | 物资标准 | string |
| Description | 是 | 需求描述 | string |
| CorpName | 是 | 需求发布方企业名 | string |
| CreateTime | 是  | 需求创建时间 | int |
| ViewsCount | 是 | 浏览量 | int |


## 3.3 查询已发布的需求(列表)详情

请求方式GET

```
https://{CloudIndustryHost}/asm/api/colla_released_demands?access_token={token}
```

查询需求列表的时候可以加过滤条件：

| 参数名 | 是否必填 | 说明 |
| ------ | ------ | ------ |
| CommonParams | 否 | 过滤条件列表，将进行与操作 |

过滤条件CommonParams说明见前文

举例说明：

1. 查询指定ID为1和10的需求（如果指定id不属于已发布或者竞标中状态，不会返回结果）
		
```
    http://host:port/api/colla_released_demands?CommonParams={"Filters":[{"name":"id","op":"in","values":["1","10"]}]}
```
	
2. 查询分类ID为2的需求

```
    http://host:port/api/colla_released_demands?CommonParams={"Filters":[{"name":"category_id","op":"==","values":["2"]}]}
```

3. 查询分类ID为2的实物需求，根据创建时间倒序排列，分页从0开始获取10项

```
    http://host:port/api/colla_released_demands?CommonParams={"Sorts":[{"name":"created_at","OrderBy":"desc"}],"Filters":[{"name":"category_id","op":"==","values":["2"]},{"name":"demand_type","op":"==","values":["realobject"]}],"Limit":10,"Offset":0}

    参数说明：
    Sorts: 排序条件
    {"name":"created_at","OrderBy":"desc"} => 需求创建时间倒序排列

    Fileters: 过滤条件
    {"name":"category_id","op":"==","values":["2"]} => 需求分类为2的需求
    {"name":"demand_type","op":"==","values":["realobject"]} => 过滤物资需求

    分页条件:
    "Limit":10,"Offset":0 => 为分页条件，Offset为起始位置，Limit为分页数量
```

4. 查询企业ID为423773857275512847的需求列表

```
    http://host:port/api/colla_released_demands?CommonParams={"Sorts":[{"name":"created_at","OrderBy":"desc"}],"Filters":[{"name":"corp_id","op":"==","values":["428797081298143301"]},{"name":"demand_type","op":"==","values":["realobject"]}],"Limit":10,"Offset":0}

    参数说明：
    Sorts: 排序条件
    {"name":"created_at","OrderBy":"desc"} => 需求创建时间倒序排列

    Fileters: 过滤条件
    {"name":"corp_id","op":"==","values":["423773857275512847"]} => 需求创建者企业为423773857275512847的需求
    {"name":"demand_type","op":"==","values":["realobject"]} => 过滤物资需求

    分页条件:
    "Limit":10,"Offset":0 => 为分页条件，Offset为起始位置，Limit为分页数量
```
    
	
返回包体示例

```json
{
    "Code": 0,
    "Msg": "",
    "DemandList": [
        {
            "ID": 36,
            "Name": "火灾识别",
            "CategoryIDList": [3],
            "CategoryList": [
                {
                    "ID": 3,
                    "Name": "最大分类1asd",
                    "Description": "测试"
                }
            ],
            "Location": "杭州",
            "Logo": "logourl",
            "Standard": "国药甲1",
            "Description": "详细说明(富文本)",
            "RequiredCount": "20000个",
            "Anonymous": false,
            "State": 3,
            "CorpID": 0,
            "CorpName": "",
            "CreateTime": 1550807412,
            "UpdateTime": 1550807412,
            "ViewsCount": 0,
            "DemandRating": 0,
            "BidCount": 0,
            "AppendixList": [
                {
                    "Name": "AAA",
                    "Url": "132.232.86.146:18081/10,01992a43273a"
                }
            ],
            "Bidding": {
                "RecvObject": "aa",
                "RecvLocation": "asa",
                "CompletionDate": 1648315396,
                "BiddingDeadline": 1648315396,
                "TimeNodeList": [
                    {
                        "Timestamp": 11,
                        "Description": "aaa"
                    }
                ]
            },
            "Budget": {
                "BudgetType": 1,
                "Min": 1,
                "Max": 2
            }
        }
    ],
    "Count": 1
}
```

| 参数名 | 基础字段 | 说明 |
| ------ | ------ | ------ |
| Code | 是 | 错误码 |
| Msg | 是 | 错误说明 |
| DemandList | 是 | 需求列表 |
| Count | 是 | 符合条件的总数量，分页的时候可能用来判断是否有更多 |

设计交互字段与字段对应关系

| 交互字段 | 对应字段 |
|:--- |:--- |
| 产品名称 | DemandList.Name |
| 产品说明 | DemandList.Description |
| 数量 | DemandList.RequiredCount |
| 标准类型 | DemandList.Standard |
| 图片示例 | DemandList.Logo |
| 创建时间 | DemandList.CreateTime |
| 到期时间 | DemandList.Bidding.BiddingDeadline |
| 浏览数量 | DemandList.ViewsCount |
| 参与人数 | DemandList.BidCount |

# 4. 留言板模块
> 评论回复相关接口

## 4.1 新建留言
> 发表评论和回复均使用该API，通过`CommentID` 可以区分是评论或回复。
权限管理：登录用户(未被禁言)均可发表留言

请求方式 POST

```
https://{CloudIndustryHost}/asm/api/msgboard/msg?access_token={{access_token}}
```

请求参数

```json
{
	"CommentID": "100",
	"Resource": "blog001",
	"Module": "community_article",
	"Content": "这是一条留言的内容",
	"ReplyUserID": "10001",
	"Private": false
}
```

**参数说明**

|字段|类型|是否必填|说明|
|---|---|---|---|
|CommentID|string|否|留言所属评论的ID。如果留言本身是评论，传"0"，或者不传均可；如果留言是回复，必传|
|Resource|string|是|被留言的资源，如文章ID，服务ID|
|Module|string|是|留言板所属模块(列表见后文)，与Resource一起唯一确定留言的对象，如工业社区的某篇文章|
|Content|string|是|留言的内容，富文本，非空|
|ReplyUserID|string|否|留言要回复的用户ID（可以理解为@某人），传"0"，或者不传，表示不回复任何人。回复时有效，评论时无效|
|Private|bool|否|是否是私密留言，不传则默认为false|

**Module列表**

- co_service: 协同制造的服务
- co_demand: 协同制造的需求

相应包体

```json
{
	"Code": 0,
	"Msg": "成功",
	"ID": "100"
}
```

**参数说明**

|字段|类型|说明|
|---|---|---|
|Code|int|错误码|
|Msg|string|错误信息|
|ID|string|留言的ID|

## 4.2 修改留言

权限管理：平台管理员 可以修改任意留言内容，其它用户只能修改自己发表的留言内容

请求方式 PUT

```
https://{CloudIndustryHost}/asm/api/msgboard/msg?access_token={{access_token}}
```

请求参数

```json
{
	"ID": "100",
	"Content": "留言的内容"
}
```

**参数说明**

|字段|类型|是否必填|说明|
|---|---|---|---|
|ID|string|否|留言的ID|
|Content|string|是|留言的内容，富文本，非空|

响应包体

```json
{
	"Code": 0,
	"Msg": "成功"
}
```

**参数说明**

|字段|类型|说明|
|---|---|---|
|Code|int|错误码|
|Msg|string|错误信息|

## 4.3 拉取评论列表

支持分页拉取某个资源下面的所有评论

权限管理：
1. 任何用户（包含登录用户和未登录用户）都可以拉取评论列表
2. 平台管理员 用户以及资源的属主（如服务/需求所属的企业管理员）的查询结果中会包含所有私密留言
3. 其它用户查询结果中不包含私密留言，除非：1. 是自己发表的私密留言 2. 是回复给自己的私密留言
4. 如果是游客，即未登录的用户，即未携带 access_token 的请求，只返回公开的留言

请求方式

```
GET
https://{CloudIndustryHost}/asm/api/msgboard/commentlist?access_token={{access_token}}&Resource={{Resource}}&Module={{Module}}&WithReply={{WithReply}}&Offset={{Offset}}&Page={{Page}}&OrderType={{OrderType}}
```

请求参数

```
GET 
https://{CloudIndustryHost}/asm/api/msgboard/commentlist?access_token={{access_token}}&Resource=blog001&Module=工业社区&WithReply=5&Offset=0&Page=10&OrderType=0
```

**参数说明**

|字段|类型|是否必填|说明|
|---|---|---|---|
|Resource|string|是|被留言的资源，如文章ID，服务ID，需求ID|
|Module|string|是|留言板所属模块，与Resource一起唯一确定留言的对象，如工业社区的某篇文章|
|WithReply|int|是|是否同时返回评论下面的回复列表，0 不返回，< 0 表示返回全部， > 0 表示最多返回多少个|
|Offset|int|是|分页的偏移|
|Page|int|是|分页的每页个数|
|OrderType|int|是|排序方式，0表示按时间逆序，1 表示按时间顺序|

> 如果 WithReply 不为0，返回的回复列表默认是按照时间顺序排序

**返回参数**

```json
{
	"Code": 0,
	"Msg": "成功",
	"TotalCnt": 99,
	"CommentList": [{
		"ID": "100",
		"Resource": "blog001",
		"Module": "community_article",
		"Content": "这是一条评论",
		"UserID": "10000",
		"UserName": "运营商",
		"CorpID": "20000",
		"Private": false,
		"CreateAt": 1568962076,
		"UpdateAt": 1568962076,
		"TotalReplyCnt": 10,
		"ReplyList": [{
			"ID": "100",
			"CommentID": "100",
			"Resource": "blog001",
			"Module": "community_article",
			"Content": "这是一条回复",
			"ReplyUserID": "10001",
			"ReplyUserName": "运营商",
			"UserID": "10000",
			"UserName": "运营商",
			"CorpID": "20000",
			"Private": false,
			"CreateAt": 1568962088,
			"UpdateAt": 1568962088
		}]
	}, {}]
}
```
**参数说明**

|字段|类型|说明|
|---|---|---|
|Code|int|错误码|
|Msg|string|错误信息|
|TotalCnt|int|总评论数量|
|CommentList|array|本次返回的评论列表|
|CommentList.ID|string|评论的ID|
|CommentList.Content|string|评论的内容|
|CommentList.UserID|string|发表该评论的用户ID|
|CommentList.UserName|string|发表该评论的用户名称|
|CommentList.CorpID|string|发表该评论的用户所属企业的ID|
|CommentList.Private|string|是否是私密留言|
|CommentList.CreateAt|int|该评论创建的时间戳|
|CommentList.UpdateAt|int|该评论最近更新的时间戳，如果发表后，没有更新过，和CreateAt相等|
|CommentList.TotalReplyCnt|int|该评论下共有多少条回复|
|CommentList.ReplyList|array|本次返回的该评论下的回复列表，列表中记录个数由 请求参数 `WithReply` 决定|
|CommentList.ReplyList.ID|string|回复的ID|
|CommentList.ReplyList.Content|string|回复的内容|
|CommentList.ReplyList.ReplyUserID|string|该条回复所回复的用户ID，"0" 表示不回复任何人|
|CommentList.ReplyList.ReplyUserName|string|该条回复所回复的用户名称，ReplyUserID 非 "0" 时有效|
|CommentList.ReplyList.UserID|string|发表该回复的用户ID|
|CommentList.ReplyList.UserName|string|发表该回复的用户名称|
|CommentList.ReplyList.CorpID|string|发表该回复的用户所属企业的ID|
|CommentList.ReplyList.Private|bool|是否是私密留言|
|CommentList.ReplyList.CreateAt|int|该回复创建的时间戳|
|CommentList.ReplyList.UpdateAt|int|该回复最近更新的时间戳，如果发表后，没有更新过，和CreateAt相等|

## 4.4 拉取回复列表

支持分页拉取某条评论下的所有回复

权限管理：
1. 任何用户（包含登录用户和未登录用户）都可以拉取回复列表
2. 平台管理员 用户以及资源的属主（如服务/需求所属的企业管理员, 文章的作者）的查询结果中会包含所有私密留言
3. 其它用户查询结果中不包含私密留言，除非：1. 是自己发表的私密留言 2. 是回复给自己的私密留言
4. 如果是游客，即未登录的用户，即未携带 access_token 的请求，只返回公开的留言

请求方式

```
GET 
https://{CloudIndustryHost}/asm/api/msgboard/replylist?access_token={{access_token}}&CommentID={{CommentID}}&Offset={{Offset}}&Page={{Page}}&OrderType={{OrderType}}
```

请求参数 (示例)

```
GET
https://{CloudIndustryHost}/asm/api/msgboard/replylist?access_token={{access_token}}&CommentID=100&Offset=0&Page=10&OrderType=1
```

**参数说明**

|字段|类型|是否必填|说明|
|---|---|---|---|
|CommentID|string|是|评论ID|
|Offset|int|是|分页的偏移|
|Page|int|是|分页的每页个数|
|OrderType|int|是|排序方式，0表示按时间逆序，1表示按时间顺序|

返回参数

```json
{
	"Code": 0,
	"Msg": "成功",
	"TotalCnt": 99,
	"ReplyList": [{
		"ID": "100",
		"CommentID": "100",
		"Resource": "blog001",
		"Module": "community_article",
		"Content": "这是一条回复",
		"ReplyUserID": "10001",
		"ReplyUserName": "运营商",
		"UserID": "10000",
		"UserName": "运营商",
		"CorpID": "20000",
		"Private": false,
		"CreateAt": 1568962088,
		"UpdateAt": 1568962088
	},{}]
}
```
**参数说明**

|字段|类型|说明|
|---|---|---|
|Code|int|错误码|
|Msg|string|错误信息|
|TotalCnt|int|总回复数量|
|ReplyList|array|本次返回的该评论下的回复列表，列表中记录个数由 请求参数 `WithReply` 决定|
|ReplyList.ID|string|回复的ID|
|ReplyList.Content|string|回复的内容|
|ReplyList.ReplyUserID|string|该条回复所回复的用户ID，"0" 表示不回复任何人|
|ReplyList.ReplyUserName|string|该条回复所回复的用户名称，ReplyUserID 非0时有效|
|ReplyList.UserID|string|发表该回复的用户ID|
|ReplyList.UserName|string|发表该回复的用户名称|
|ReplyList.CorpID|string|发表该回复的用户所属企业的ID|
|ReplyList.Private|bool|是否是私密留言|
|ReplyList.CreateAt|int|该回复创建的时间戳|
|ReplyList.UpdateAt|int|该回复最近更新的时间戳，如果发表后，没有更新过，和CreateAt相等|


## 4.5 拉取指定留言的信息

拉取指定的留言信息，包括某条评论的信息，或者某条回复的信息

权限管理：
1. 任何用户（包含登录用户和未登录用户）都可以拉取留言内容
2. 平台管理员 用户以及资源的属主（如服务/需求所属的企业管理员, 文章的作者）可以拉取所有私密的留言内容
3. 其它用户只能拉取公开的留言以及 1. 自己发表的私密留言 2. 回复给自己的私密留言
4. 如果是游客，即未登录的用户，即未携带 access_token 的请求，只能拉取公开的留言

请求方式

```
GET 
https://{CloudIndustryHost}/asm/api/msgboard/msg?access_token={{access_token}}&ID={{ID}}
```

请求参数

```
GET 
https://{CloudIndustryHost}/asm/api/msgboard/msg?access_token={{access_token}}&ID=100
```

**参数说明**

|字段|类型|是否必填|说明|
|---|---|---|---|
|ID|string|是|留言的ID|

返回参数

```json
{
	"Code": 0,
	"Msg": "成功",
    "TotalCnt": 1,
	"Records": [{
		"ID": "100",
		"CommentID": "100",
		"Resource": "blog001",
		"Module": "community_article",
		"Content": "这是一条回复",
		"ReplyUserID": "10001",
		"ReplyUserName": "运营商",
		"UserID": "10000",
		"UserName": "运营商",
		"CorpID": "20000",
		"Private": false,
		"CreateAt": 1568962088,
		"UpdateAt": 1568962088
	}]
}
```
**参数说明**

|字段|类型|说明|
|---|---|---|
|Code|int|错误码|
|Msg|string|错误信息|
|TotalCnt|int|总数量，该接口最大为1（为了方便后续拓展，故增加该字段）|
|Records|array|留言的列表，该接口中列表最多只会有一个元素（为了方便后续拓展，故设计成array）|
|Records.ID|string|留言的ID|
|Records.CommentID|string|留言所属评论的ID，如果留言本身是评论，则该字段为 "0" |
|Records.Content|string|留言的内容|
|Records.ReplyUserID|string|留言要回复的用户ID（可以理解为@某人）, "0" 表示不回复任何人。仅在留言是回复时有效|
|Records.ReplyUserName|string|留言要回复的用户名称（可以理解为@某人）, ReplyUserID 非 "0" 时有效|
|Records.UserID|string|发表该留言的用户ID|
|Records.UserName|string|发表该留言的用户名称|
|Records.CorpID|string|发表该留言的用户所属企业的ID|
|Records.Private|bool|是否是私密留言|
|Records.CreateAt|int|该留言创建的时间戳|
|Records.UpdateAt|int|该留言最近更新的时间戳，如果发表后，没有更新过，和CreateAt相等|

> 可以通过CommentID是否为"0" 来判断该留言是评论还是回复

## 4.6 删除指定的留言

删除指定的留言
如果是评论，会同步删除该评论下所有的回复
如果是回复，只会删除该条回复

权限管理：
1. SP能删除任意留言
2. 其它用户只可以删除自己发表的留言

请求方式

```
DELETE 
https://{CloudIndustryHost}/asm/api/msgboard/msg?access_token={{access_token}}&ID={{ID}}
```

请求参数

```
DELETE 
https://{CloudIndustryHost}/asm/api/msgboard/msg?access_token={{access_token}}&ID=100
```

**参数说明**

|字段|类型|是否必填|说明|
|---|---|---|---|
|ID|string|是|留言的ID|

返回参数

```json
{
	"Code": 0,
	"Msg": "成功"
}
```
**参数说明**

|字段|类型|说明|
|---|---|---|
|Code|int|错误码|
|Msg|string|错误信息|