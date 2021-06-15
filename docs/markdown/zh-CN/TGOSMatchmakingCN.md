## TGOS Matchmaking Introduction

### 1. Matchmaking 是什么

TGOS Matchmaking功能帮助游戏将符合一定匹配规则的玩家组成对局。TGOS Matchmaking 整合玩家查找、DS放置等流程，支持单人/多人发起匹配、取消匹配、查询匹配状态等功能。

### 2. 关键词解析

**匹配规则**：对匹配计算规则及依赖数据的一组描述脚本，可被多个匹配配置复用。

**匹配配置**：集合匹配规则、DS需求等配置信息的对象。客户端发起匹配时必须制定使用的匹配配置。每个匹配配置需要关联唯一的匹配规则。

### 3. 模块交互图

游戏客户端通过TGOS Client SDK访问Matchmaking服务。主要涉及两类调用，通过Http协议访问的invoke接口及通过WebSocket长连接实现的推送事件消息；Matchmaking Service支持从PlayerManagement获取PlayerProperties数据；游戏DS通过TGOS DS SDK协助DS Management完成DS的放置工作。

<img src="/markdown/TGOS-MatchMaking/1600680137699.png" />

### 4. 核心功能及接口

#### 4.1 发起匹配

发起包含一名或多名玩家的匹配请求，用户可在请求中指定匹配玩家的队伍、属性及延迟信息。

```C++
/*
* @brief Start a matchmaking request.
* @param configuration_name Name of matchmaking configuration
* @param player_info_list Member's detail info in matchmaking
* @param callback result callback for the function
* @return
*/
virtual void StartMatchmaking(
    const std::string& configuration_name,
    std::vector<http_model::GamePlayerInfo> player_info_list,
	HttpCallback<http_model::StartMatchmakingInfo> call_back = nullptr) = 0;
```

```c++
struct GamePlayerInfo {
    std::string name;
    std::string id;
    std::string team;
    int custom_player_status;
    std::string custom_profile;
    std::vector<MatchAttributeInfo> match_attributes;
    std::vector<RegionLatencyInfo> player_latency;
};

struct MatchAttributeInfo {
    std::string name;
    int type;
    std::string value;
};

struct LatencyInMsInfo {
    std::string region_name;
    long latency;
};

struct StartMatchmakingInfo {
    std::string ticket_id;
};
```

#### 4.2 匹配请求确认

当匹配请求中包含多名玩家时，MatchServer会向所有请求成员发起一次确认通知，收到通知的玩家可选择加入本次匹配请求（JoinMatchmaking）或拒绝本次匹配请求（RejectMatchmaking）；任何一名成员拒绝匹配请求，均会导致该匹配请求终止。

此设计的原因有二：

- 我们遵循这样一个设计原则，所有多他人产生影响的多人操作均需参与人确认
- 我们预设存在匹配发起人无法获取的匹配成员数据，匹配成员可在接受匹配请求时填充这些数据

**Remarks**

如果开发者不希望该确认阶段被玩家感知到，可以在收到NotifyMatchmakingPreparation后直接调用JoinMatchmaking以接受本次多人匹配请求。

**功能时序**

```sequence
participant Player1
participant Player2
participant MatchServer

Player1->MatchServer:StartMatching\n(Player1+Player2)

MatchServer-->Player2:NotifyMatchmakingPreparation
Player2->MatchServer:JoinMatchmaking\n(With Player2's GamePlayerinfo)
MatchServer-->Player1:Notify:MatchProgress
```

**JoinMatchmaking**

通过该接口加入一个由他人发起的包含此玩家的多人匹配请求。

```C++
/*
* @brief Accept to join a multiplayer matchmaking request.
* @param ticket_id Ticket id for the matchmaking request
* @param player_info Player's detail info
* @param reason The reason for the join active, not necessary
* @param callback result callback for the function
* @return
*/
virtual void JoinMatchmaking(
    const std::string& ticket_id,
    const http_model::GamePlayerInfo& player_info,
    const std::string& reason = "",
    HttpCallback<Void> call_back = nullptr) = 0;
```

**RejectMatchmaking**

```C++
/*
* @brief Reject a multiplayer matchmaking request.
* @param ticket_id Ticket id for the matchmaking request
* @param player_info Player's detail info, not necessary
* @param reason The reason for the join action, not necessary
* @param callback result callback for the function
* @return
*/
virtual void RejectMatchmaking(
    const std::string& ticket_id,
    const http_model::GamePlayerInfo& player_info,
    const std::string& reason = "",
    HttpCallback<Void> call_back = nullptr) = 0;
```

| Name      | Type                 | Description        |
| --------- | -------------------- | ------------------ |
| ticket_id | string               | 可标识匹配请求的ID |
| reason    | string               | 拒绝的原因         |
| call_back | HttpCallback\<Void\> | 接口完成回调       |

#### 4.3 查询匹配进度信息

可通过`DescribeMatchmaking`接口查询匹配进度信息。

```c++
/*
* @brief Get detail process info of a matchmaking request.
* @param ticket_id Ticket id for the matchmaking request
* @param callback result callback for the function
* @return
*/
virtual void DescribeMatchmaking(
    std::string ticket_id,
    OnMatchmakingProcess call_back = nullptr) = 0;
```

```C++
enum class MatchmakingStatus : int {
	Normal = 0,
	Starting,		// Matchmaking is starting
	Preparing,		// The mutilplayer matchmaking's preparation phase
	Searching,		// Search players
	Placing,		// Placing battle session
	Activing,		// Activing battle session
	Completed,		// Matchmaking complete
	Terminated,		// Matchmaking is terminated
	Canceled,		// Matchmaking is canceled by player
	Error			// Error
};
```

#### 4.4 取消匹配

可通过`CancelMatchmaking`接口查询匹配进度信息。

```C++
/*
* @brief Cancel a matchmaking request.
* @param ticket_id Ticket id for the matchmaking request
* @param reason The reason for the cancel action, not necessary
* @param callback result callback for the function
* @return
*/
void CancelMatchmaking(
    const std::string& ticket_id,
    const std::string& reason = "",
    HttpCallback<http_model::CancelMatchmakingInfo> callback = nullptr);
```

#### 4.5 监听管理

可通过`SetMatchmakingObserver`接口注册匹配进度事件的监听者。

```C++
/*
* @brief Set the observer of matchmaking process
* @param observer event observer，input nullprt to remove observer
*/
virtual void SetMatchmakingObserver(OnMatchmakingProcess observer) = 0;
```

可通过`SetMatchmakingPreparationObserver`接口注册多人匹配确认事件的监听者。

```c++
/*
* @brief Set the observer of multiplayer matchmaking preparation
* @param observer event observer，input nullprt to remove observer
*/
virtual void SetMatchmakingPreparationObserver(OnMatchmakingProcess observer) = 0;
```

#### 4.6 匹配进度信息通知

通知消息格式同 DescribeMatchmaking。

### 5. 匹配规则介绍

#### 5.1 一个例子

```json
{
    // playerAttributes 小节描述了匹配规则中使用的 attributes of players
    "playerAttributes":[
        {
            "name": "kda",
            "key":"kda"			//  Attribute from Player properties which key is "kda"
            "type": "number"
            "default": 10
        },
    ],
    // teams 小节描述对局的 team 结构
    // 本例中的team结构为 5vs5 
    "teams":[
        {
            "name":"Darkness",
            "minPlayers":5,
            "maxPlayers":5,
            "number":1,
        },
        {
            "name":"Light",
            "minPlayers":5,
            "maxPlayers":5,
            "number":1,
        }
    ],
    // rules 小节可添加多条规则描述，TGOS 根据这些规则搜索玩家
    // 下面内容的含义为：各team成员平均kda与所有成员平均kda的绝对差值不大于10，不小于5
    "rules": [
        {
            "name": "distanceRuleExample",
            "type": "distanceRule",
            // List<Avg kda of players in team Darkness,
            // 		Avg kda of players in team Light>
            "measurements": [
                "avg(teams[*].players.playerAttributes[kda])"
            ],
            "referenceValue": "avg(flatten(teams[*].players.playerAttributes[kda]))",
            "minDistance":5,
            "maxDistance":10,
        }
    ]
    // expansions 小节为对 rules 的扩展
    // 下面内容的含义为：
    //		(1). 等待5s，若无匹配结果产生，将rules[distanceRuleExample].minDistance 修改为4，
    //		将rules[distanceRuleExample].maxDistance修改为15
    // 		(2). 等待10s，若无匹配结果产生，将rules[distanceRuleExample].maxDistance修改为20
     "expansions": [
         {
            "target": "rules[distanceRuleExample].minDistance",
            "steps": [{
                "waitTimeSeconds": 5,
                "value": 4
            }]
        },
        {
            "target": "rules[distanceRuleExample].maxDistance",
            "steps": [{
                "waitTimeSeconds": 5,
                "value": 15
            },{
                "waitTimeSeconds": 10,
                "value": 20
        }
    ]
}
```

#### 5.2 规则总览

TGOS支持通过`Json`结构的脚本描述匹配规则。

**脚本结构**

| Field            | Description                                                 |
| ---------------- | ----------------------------------------------------------- |
| version          | 规则集脚本版本                                              |
| playerAttributes | 玩家信息，这些信息可在rules中使用                           |
| teams            | 对局team的约束，这些约束信息可在rules中使用                 |
| rules            | 若干匹配约束条件，其中使用 playerAttributes 及 teams 的数据 |
| expansions       | 对 rules 的逐次迭代的扩展                                   |

**e.g**

```json
{
    "playerAttributes":[
        ...
    ],
    "teams":[
        ...
    ],
    "rules":[
        ...
    ],
    "expansions":[
        ...
    ],
}
```

#### 5.3 playerAttributes

该字包含对多组玩家信息的描述，TGOS 通过这些描述知晓该从哪里获取哪些数据。

**脚本结构**

| 字段    | 描述                                 | 是否必填 | 说明                                    |
| ------- | ------------------------------------ | -------- | --------------------------------------- |
| name    | 属性名称                             | 是       | 长度32的string。取值范围a~z,A~Z,0~9,_是 |
| type    | 属性类型                             | 是       | 取值“number”, “string”                  |
| default | 属性默认值                           | 否       | 必须与该属性的类型一致                  |
| key     | Player properties<br /> 中定义的 key | 否       | TGOS Player properties 中玩家数据的key  |

**数据来源**

- 来源一，如果`key`字段为空，意味着该属性取自匹配请求的`match_attributes`字段中，name一致的item
- 来源二，如果`key`字段不为空，意味着该数据来自Player properties中，key一致的item

**e.g**

```json
    "playerAttributes": [
        {
            "name": "age", // Attribute from match_attributes中key为 which key is "age"
            "type": "number"
            "default": 10
        },
        {
            "name": "skill", // Attribute from match_attributes中key为 which key is "skill"
            "type": "number"
        },
        {
            "name": "kda", // Attribute from Player properties which key is "kda"
            "key":"kda"
            "type": "number"
            "default": 10
        },
    ],
```

#### 5.4 teams

**脚本结构**

| 字段       | 描述         | 是否必填 | 说明                                         |
| ---------- | ------------ | -------- | -------------------------------------------- |
| name       | 队伍名称     | 是       | 长度32的string。取值范围a~z,A~Z,0~9,_        |
| minPlayers | 最小玩家数量 | 是       | 0-40内整数                                   |
| maxPlayers | 最大玩家数量 | 是       | 0-40内整数，maxPlayers必须大于等于minPlayers |
| number     | 队伍数量     | 否       | 0-40内整数。默认值为1                        |

**e.g**

```json
// 5vs5 fair play
"teams":[
    {
        "name":"Darkness",
        "minPlayers":5,
        "maxPlayers":5,
        "number":1,
    },
    {
        "name":"Light",
        "minPlayers":5,
        "maxPlayers":5,
        "number":1,
    }
]

// 1vs1vs1vs1vs1 Solo
"teams":[
    {
        "name":"solo_team",
        "minPlayers":1,
        "maxPlayers":1,
        "number":5,
    }
]
```

#### 5.5 rules

**脚本结构**

| 字段        | 描述     | 是否必填 | 说明                                                       |
| ----------- | -------- | -------- | ---------------------------------------------------------- |
| name        | 规则名称 | 是       | 长度32的string。取值范围a~z,A~Z,0~9,_                      |
| description | 规则描述 | 否       | 长度128的string                                            |
| type        | 规则类型 | 是       | 有效取值“distanceRule”, “comparisonRule” ,   “latencyRule” |

**约束**

一个规则集的规则条数不超过10条。

#### 5.6 规则表达式

**语法规范**

下列语句展示了如何标识team及team中的玩家。

```json
// 含义：
// 		Darkness team
// 结果类型：
// 		Team
teams[Darkness]

// 含义：
// 		All teams
// 结果类型：
// 		List<Team>
teams[*]

// 含义：
// 		Players in Darkness team
// 结果类型：
// 		List<Player>
teams[Darkness].players
      
// 含义：
// 		All players group by team
// 结果类型：
// 		List<List<Player>>
teams[*].players
```

下列语句展示了如何获取玩家属性数据。

```json
// 含义：
// 		Playerid of players in Darkness Team
// 结果类型：
// 		List<string>
teams[Darkness].players[playerid]

// 含义：
// 		List of attribute named skill in Darkness Team
// 结果类型：
// 		List<number>
teams[Darkness].players.playerAttributes[skill]

// 含义：
// 		Attribute named skill of all players(Group by team)
// 结果类型：
// 		List<List<number>>
teams[*].players.playerAttributes[skill]
```

**算法函数**

| 函数名  | 输入          | 含义                                         | 输出    |
| ------- | ------------- | -------------------------------------------- | ------- |
| avg     | List<number>  | 获取列表中所有数字的平均值。                 | number  |
| flatten | List<List<?>> | 将嵌套列表的集合变成包含所有元素的单个列表。 | List<?> |

**e.g of flatten**

```json
ListA = ["a", "b", "c"]

ListB = ["d", "e", "f"]

ListC = [ListA, ListB]

flatten(ListC) = ["a", "b", "c", "d", "e", "f"]

// 含义：
//		List of attribute named kda of all players
// 结果类型：
//		List<number>
flatten(teams[*].playerAttributes[kda])
```

#### 5.7 规则类型

**类型1. 距离规则**

| 属性名称         | 属性描述                             | 是否必填                           | 取值约束                                                     |
| ---------------- | ------------------------------------ | ---------------------------------- | ------------------------------------------------------------ |
| name             | 规则名称                             | 是                                 | 长度32的string。取值范围a~z,A~Z,0~9,_。在同一个规则集内不能重复 |
| type             | 规则类型                             | 是                                 | distanceRule                                                 |
| description      | 规则描述                             | 否                                 | String，长度256                                              |
| measurements     | 测量值。用于与referenceValue比较距离 | 是                                 | 计算结果为List<number>或List<List<number>>的表达式<br />可支持多个表达式<br />number保留两位小数 |
| referenceValue   | 比较值                               | 是                                 | Number或计算结果为number的表达式。number保留两位小数。可作为扩展目标 |
| minDistance      | 最小距离差                           | minDistance和maxDistance至少填一个 | measurements中计算出的每一个number与referenceValue比较，可允许差距的最小值。取值范围0-99999之间的number，number保留两位小数。可作为扩展目标 |
| maxDistance      | 最大距离差                           | minDistance和maxDistance至少填一个 | measurements中计算出的每一个number与referenceValue比较，可允许差距的最大值。取值范围0-99999之间的number，number保留两位小数。可作为扩展目标 |
| partyAggregation | 处理多玩家请求的方式                 | 否                                 | String,有效值为使用发出请求玩家的“min”, “max”, “avg”。默认平均值 |

```json
// 含义：
//		The average of the skill of each team player, and the average of the skill of all // 	  players in the world is not less than 5, not more than 10
"rules": [
    {
        "name": "distanceRuleExample",
        "type": "distanceRule",
        "measurements": [
            "avg(teams[*].players.skill)"
        ],
        "referenceValue": "avg(flatten(teams[*].players.playerAttributes[skill]))",
        "minDistance":5,
        "maxDistance":10,
    }]
```

**类型2. 比较规则**

| 属性名称       | 属性描述                     | 是否必填 | 取值约束                                                     |
| -------------- | ---------------------------- | -------- | ------------------------------------------------------------ |
| name           | 规则名称                     | 是       | 长度32的string。取值范围a~z,A~Z,0~9,_。在同一个规则集内不能重复 |
| description    | 规则描述                     | 否       | String，长度256                                              |
| type           | 规则类型                     | 是       | comparisonRule                                               |
| measurements   | 规则表达式                   | 是       | 计算结果为List<?>的表达式，可以是List<number>或List<string>。暂时只支持1个表达式。 |
|                |                              |          | 如果referenceValue有定义，则将List<?>中的每一个元素与referenceValue比较； |
|                |                              |          | 如果referenceValue未定义，则在List<?>中的每一个元素之间比较，此时operation只能取值“=”或“！=”。 |
| referenceValue | 用于与规则表达式进行比较的值 | 否       | 计算结果与measurement的元素类型相同的表达式，如number，string。可作为扩展的目标 |
| operation      | 比较判断符                   | 是       | 合法取值：“=”，“ !=”，“ <”，“ <=”，“   >”， “ >=”( 这些符号都可以比较number或string) 。可作为扩展目标 |

**类型3. 延迟规则**

| 属性名称          | 属性描述                          | 是否必填 | 取值约束                                                     |
| ----------------- | --------------------------------- | -------- | ------------------------------------------------------------ |
| name              | 规则名称                          | 是       | 长度32的string。取值范围a~z,A~Z,0~9,_。在同一个规则集内不能重复 |
| description       | 规则描述                          | 否       | String，长度256                                              |
| type              | 规则类型                          | 是       | comparisonRule                                               |
| maxLatency        | 最大延时                          | 是       | 单位为ms；<br />取值范围0~999999；<br />可作为扩展目标       |
| maxDistance       | 延时与distanceReference的最大差值 | 否       | 单位为ms；<br />取值范围0~999999；<br />可作为扩展目标       |
| distanceReference | 用于与比较                        | 否       | 单位为ms；<br />取值范围0~999999；<br />可作为扩展目标       |
| partyAggregation  | 处理多玩家请求的方式              | 否       | String,有效值为使用发出请求玩家的“min”, “max”, “avg”。默认为“avg” |

```json
// 含义：
//		The maximum delay from the player to the region does not exceed 150ms
"rules": [{
        "name": "laytency_rule_example",//
        "type": "latencyRule",
		"maxLatency": 150
    }]
```

#### 5.8 规则扩展

匹配规则支持针对部分字段的约束进行扩展，以实现逐渐放宽的匹配约束。

**规则扩展约束**

- 一个匹配规则集中的expansion，全部target的数量不超过5条
- 一个target的steps数量不超过10条

**e.g**

```json
// 含义：
//		After waiting 5s for no result, modify the value of minDistance to 4
//		Continue to wait for 5 seconds without results, modify the value of maxDistance //		to 15
//		Continue to wait for 10 seconds without results, modify the value of maxDistance //		 to 20
"rules": [
    {
        "name": "distanceRuleExample",
        "type": "distanceRule",
        "measurements": [
            "avg(teams[*].players.playerAttributes[skill])"
        ],
        "referenceValue": "avg(flatten(teams[*].players.playerAttributes[skill]))",
        "minDistance":5,
        "maxDistance":10,
    }
]
 "expansions": [
     {
        "target": "rules[distanceRuleExample].minDistance",
        "steps": [{
            "waitTimeSeconds": 5,
            "value": 4
        }]
	},
    {
        "target": "rules[distanceRuleExample].maxDistance",
        "steps": [{
            "waitTimeSeconds": 5,
            "value": 15
        },{
            "waitTimeSeconds": 10,
            "value": 20
	}
]

```

### 6. Web Portal 核心功能

**创建匹配规则功能概念图：**

![1600674530524](/markdown/TGOS-MatchMaking/1600674530524.png)

**创建匹配配置功能概念图：**

![1600674563736](/markdown/TGOS-MatchMaking/1600674563736.png)

