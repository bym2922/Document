# 玩家鉴权

## 1. 什么是玩家鉴权

玩家鉴权是验证玩家访问TGOS服务权利的过程。TGOS本身不提供实质的账号服务系统，通过对接第三方账号服务系统（例如Tencent IntlSDK），在后端验证账号票据确认玩家账号有效性，进而让玩家客户端获得访问TGOS服务的权利。

## 2. 关键词释义

- **Account Service**: 账号服务系统（例如Tencent IntlSDK）
- **Tencent IntlSDK**：腾讯为海外游戏提供的发行期技术解决方案，包括账号登录、事件上报等功能
- **open_id**: Account Service提供的用户标识
- **token**: open_id对应的有效票据，可能需要通过Account Service续票
- **player_id**： TGOS体系提供的玩家标识，全局唯一
- **player_ticket**: 玩家鉴权有效票据，TGOS自动续票（SDK已屏蔽，开发者无需关心）
- **player_session**: 从玩家TGOS鉴权成功，到玩家退出或者鉴权失效的这一个会话过程。一旦玩家会话失效，游戏客户端将不能从TGOS获得服务。
- **player_session data**: 与player_session关联的一组公开的kv-data，可用于存储玩家的在线状态，网络等，这组数据是非持久的，玩家会话失效后，数据被移除。（后期实现）
- **fake account service**: 模拟Account Service，帮助开发商在未接入实际的Account Service前，跑通所有依赖发行账号服务的TGOS功能。
- **account_chn**: 账号渠道，0: fake account service, 1: Tencent IntlSDK

## 3. 鉴权时序图

```sequence
###Title:Authentication sequence diagram
Game Client->Account Service: login
Account Service-->Game Client: return: open_id & token
Game Client->TGOS SDK: StartPlayerSession(open_id, token)
TGOS SDK->>TGOS Backend: StartPlayerSession(open_id, token, account_chn)
TGOS Backend->>Account Service: Auth(open_id, token)
Account Service--> TGOS Backend: return:success or failed
TGOS Backend-->TGOS SDK: success:player_id & player_ticket
TGOS SDK-->Game Client: success: player_id, failed: error msg
```

## 4. 关键接口说明

- player session事件监听

  ```c++
  /**
   * Set callback for player-session event
   *
   * @param callback  Result callback for the function
   */
  virtual void SetEventCallback(std::function<void(PlayerSessionEvt evt, const std::string& msg)> callback) = 0;
  
  //---------------------------------------------------------------
  // Purpose: define player-session event in its lifetime.
  //---------------------------------------------------------------
  enum class PlayerSessionEvt : int {
      // triggered when player-session is started successfully
  	SessionStart = 0, 
      // triggered when the network is abnormal
      // call RestartPlayerSession() may help.
      SessionExpired_HeartbeatFailed = 1,
      // triggered when the account token expired
      // Re login account and start a new player-session may help.
      SessionExpired_TokenExpired = 2,
      // may triggered by:
      // 1. the account login on other devices
      // 2. the account is cancelled
      // 3. the account is restricted by webportal
      // It is recommended to inform players to check account status
      SessionExpired_Kickout = 3,  
      // triggered when player-session ends
      SessionEnd
  };
  ```

- 启动会话

  - open_id和token可以在登录成功后从Account Service返回信息得到；

  - title_region_id可以从选区返回的信息得到；在web portal建立区服时可确认title_region_id

  ```c++
  /**
   * Start a new player-session to gain access to TGOS service
   *
   * @param open_id  Open id of the logged in account
   * @param token  Token of the open id
   * @param title_region_id  Region the player wants to visit
   * @param callback  Result callback for the function
   */
  virtual void StartPlayerSession(const std::string& open_id, const std::string& token,
                                  const std::string& title_region_id, 
                                  HttpCallback<AuthenticationInfo> callback) = 0;
  ```
  
```C++
  struct AuthenticationInfo {
      bool is_first_login; // is this account playing the game for the first time 
      std::string player_id; // palyer id corresponding to the logined account
      std::string session_id;  // player's login session id
  };
  ```
  
- 刷新账号票据

  当Account Service的账号票据有更新时，要通过下面的接口通知TGOS。

  ```c++
  /**
   * Inform TGOS of the latest account token.
   *
   * @param open_id  Open id of the logged in account
   * @param token  Token of the open id
   * @param callback  Result callback for the function
   */
  virtual void UpdateAccountToken(const std::string& open_id, const std::string& token,
                                  HttpCallback<Void> callback) = 0;
  ```
  

  
- 玩家会话失效（网络异常、账号票据异常等）后的重新激活

  ```C++
  /**
   * Try to make the player-session be valid again
   */
  virtual void RestartPlayerSession() = 0;
  ```
  
- 结束会话

  ```c++
  /**
   * Terminate player-session, close access to TGOS service
   *
   * @param callback  Result callback for the function
   */
  virtual void EndPlayerSession(HttpCallback<Void> callback) = 0;
  ```

  

## 5. 使用说明

- 可以通过下面的接口获得IPlayerAuth实例：

  ```c++
  IPlayerAuth *PlayerAuth();
  ```

- 监听玩家会话事件

  ```c++
  void SetEventCallback(std::function<void(
      PlayerSessionEvt evt, const std::string& msg)> callback);
  ```

- 当游戏客户端通过 **Account Service**（例如Tencent IntlSDK）完成账号登录后，即可启动玩家会话：

  ```c++
  void StartPlayerSession(const std::string& open_id,
                          const std::string& token,
                          const std::string& title_region_id,
                          HttpCallback<AuthenticationInfo> callback);
  ```

- 当账号的token被更新后（与**Account Service**有关），要通过下面的接口把新token通知TGOS

  ```c++
  void UpdateAccountToken(const std::string& open_id,
                          const std::string& token,
                          HttpCallback<Void> callback);
  ```

- 如果玩家会话失效（网络异常，账号票据异常等，原因见PlayerSessionEvt的枚举），开发者可以考虑尝试重新使玩家会话有效，如果是账号票据异常，可以尝试重新登录账号再重新启动新会话，如果是其他异常，也可以放弃（TGOS内部其实已经做了限时限次的重试）。一旦玩家会话失效，游戏客户端将不能从TGOS获得服务。

  ```C++
  void RestartPlayerSession();
  ```

- 当玩家要结束玩家会话，需要调用下面IPlayerAuth的接口：

  ```C++
  void EndPlayerSession(HttpCallback<Void> callback);
  ```

- 如果玩家要切换账号，请使用下面的流程完成切换：

  - 调用IPlayerAuth::EndPlayerSession()接口结束之前的玩家会话；
  - 通过 **Account Service** 切换账号并登录成功；
  - 调用IPlayerAuth::StartPlayerSession()启动新的玩家会话；

## 6. Fake Account System

模拟发行账号登录的鉴权服务，提供相关的接口和流程，帮助开发商在未接入实际的发行账号服务前，跑通所有依赖发行账号服务的外部功能，它是发行账号服务的极简化功能版本的替代品。

可以通过下面的接口获得IFakeAccountSystem实例：

```C++
IFakeAccountSystem* FakeAccountSystem();
```

它主要通过两个接口完成账号相关服务：

- 账号登录

  ```c++
  /*
   * @brief  account logins.
   * @param  callback  result callback for the function
   * @return  
   */
  void Login(OnLoginResult callback);
  ```

- 刷新票据

  ```c++
  /*
   * @brief  refresh account token before token is expired.
   * @param  token  last token
   * @param  callback  result callback for the function
   * @return  
   */
  void RefreshToken(const std::string &token, OnRefreshTokenResult callback)
  ```

  



