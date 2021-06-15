# TGOS Party Introduction

### 1. Party是什么？

TGOS Party为游戏提供完整的组队功能实现，包括组队流程、事件通知和灵活的用户交互支持。一个Party由一个队长和多个成员玩家组成，任何玩家都有权限创建Party，创建者默认是队长，队长也可以将队长身份转移到某个成员。

### 2. 关键词释义

* Party：游戏里的一个队伍，英文名称为Party
* Leader：创建Party的人即为队长
* Member：Party成员
* Party Name：创建Party时取的Party的名字，可重复

### 3. 服务架构图

游戏通过TGOS Client SDK访问Party服务，主要涉及两类调用，一类是invoke，主要通过Http协议访问；另一类是Client Event，是Push Service向TGOS Client SDK推送的事件消息，通过Web Socket长连接完成，这个连接是登录成功后，TGOS Client SDK自动建立并维护的。

![1600416838696](/markdown/TGOS-Party/1600416838696.png)

### 4. 主要功能和规则

1. **创建Party**

   任何玩家都可以创建Party，有以下方式：

   * 玩家直接创建一个Party
   * 玩家邀请另一个玩家组成Party

   注：一个人同时只能创建一个Party

2. **Party邀请**

   Leader或Member可以邀请他人加入Party，邀请发生在以下情形：

   * 一个玩家在没有加入任何Party的情况下，邀请他人组成一个新的Party，该玩家也会成为新Party的Leader
   * 一个Party中的Leader或Member，邀请他人加入Party
   * 一个人同时只能在一个Party中

   ![1600417403600](/markdown/TGOS-Party/1600417403600.png)

3. **Leader特权**

   * 踢人kick：Leader可以随时kick任何一个其他Member
   * 解散Party
   * 可以转让Leader给其他某个成员
   
4. **Party解散**

   以下情况Party会解散：

   * Leader主动解散
   * 当有人离开Party，若剩下人数为0时，Party自动解散

   ![1600418542252](/markdown/TGOS-Party/1600418542252.png)

5. **离开Party**

   Party中的任何人可离开Party，当Leader离开Party后，TGOS自动授权某个其他成员为Leader

   任何人离线后，自动离开Party

   ![1600418557494](/markdown/TGOS-Party/1600418557494.png)

9. **发起Matchmaking**

   基于设计考虑，Party与Matchmaking完全解耦，可以使用Matchmaking提供的team模式，将Party所有玩家信息放到一个data中，以参数传入Matchmaking Request

7. **加入Lobby**

   基于设计考虑，Party与Lobby完全独立，可以使用Lobby提供的team模式，将Party所有玩家信息放到一个data中，以参数传入Lobby Enter接口

8. **聊天功能**

   结合Chat Service，游戏可以实现Party内聊天的功能（后期实现）

9. **实时Party查询**

   查询当前活跃的Party

   * Party List：当前活跃的Party List
   * Party Info：某个Party的信息：包含Party Id、Name、Leader、Members、Created Time、Data（序列化后的数据）

10. **历史记录**

    Party解散后，系统自动生成一条历史记录（落地存储），该记录包含以下信息：

    * Leader Player Id，过程中可能有多个Leader，此处需要将每个Leader的信息记录下来
    * Member Player Ids，过程中Member可能会变动，此处需要将Member的变动信息记录下来
    * Party Created Time和Destroy Time
    * Party Id、Name
    * 序列化的Party Data


### 4. 关键属性和数据

| 属性                 | 属性说明          | 备注                  |
| -------------------- | ----------------- | --------------------- |
| **Party Id**         | Party的标识       | 全局唯一，如使用GUID  |
| **Name**             | Party的名字       | 创建Party设置，不唯一 |
| **Leader Player Id** | Leader的player id |                       |
| **Created Time**     | 创建时间          |                       |
| **Dissmissed Time**  | 解散时间          |                       |
| **Members**          | 成员列表          | 包含leader            |
### 5. 关键接口和事件

#### 5.1 客户端接口

| 接口名称                                     | 描述                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| CreateParty(name,invitee_player_ids[option]) | 创建Party，invitee_player_ids为空时是自己创建Party，不为空时是邀请他人(可能是多人)创建一个Party；他人是否接受邀请不影响Party的创建 |
| GetPartyInfo(party_id)                       | 获取Party的信息（name，成员列表，party_data等）              |
| InviteParty(invitee_player_ids)              | 邀请他人(多人)加入Party                                      |
| JoinParty(party_id)                          | 加入某个Party                                                |
| LeaveParty(party_id)                         | 退出某个Party                                                |
| DismissParty(party_id)                       | 【队长特权】解散某个Party                                    |
| KickPartyMember(party_id, player_id)         | 【队长特权】踢掉某人                                         |
| TransferPartyLeader(party_id, player_id)     | 【队长特权】转移Leader权限给其他Member                       |
|                                              |                                                              |

#### 5.2 客户端事件

| 时间名称                                                     | 描述                                                        |
| ------------------------------------------------------------ | ----------------------------------------------------------- |
| OnPartyInviting(party_id, inviter_player_id)                 | 玩家收到邀请通知                                            |
| OnPartyMemberJoined(party_id,member_player_id)               | 有人加入party后的事件                                       |
| OnPartyMemberLeave(party, reason)                            | 有人离开party，reason提示离开原因（主动离开，被踢，离线等） |
| OnPartyDismissed(party_id)                                   | party被解散                                                 |
| OnPartyLeaderTransfered(party_id, previous_leader_player_id) | Leader权限被转让给自己                                      |
|                                                              |                                                             |



### 6. 关键功能描述

#### 6.1 发起Matchmaking

**流程：**

Leader可以发起Matchmaking，操作流程是：

	* Leader通过游戏UI，进入发起Matchmaking流程
	* Leader所在客户端，获取每个成员的player_id，构造player_id list
	* Leader所在客户端，以player_id list作为核心参数，调用Matchmaking的StartMatchmaking接口
	* Member所在客户端，马上会收到通知，指示Leader发起了匹配
	* Member所在客户端，调用Matchmaking的JoinMatchmaking接口，并进入匹配界面
	* Matchmaking成功后，所有玩家会收到通知，提示进入对局

#### 6.2 加入Lobby

后期实现

#### 6.3 成员交互

后期实现

### 7. Web Portal功能

* 查询活跃的Party
* 查询Party流水记录

Portal交互界面概念图：

![1600419116609](/markdown/TGOS-Party/1600419116609.png)