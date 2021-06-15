## 1. 什么是玩家资料

玩家资料是与玩家绑定的一组数据集合，主要分为3部分：

- 玩家基础信息：由TGOS定制的玩家固有信息，所有玩家都有该信息；
- 玩家属性集合：由开发者定制的玩家固有信息，所有玩家都有该信息；
- 玩家数据集合：开发者随机为玩家赋予的信息，仅部分玩家拥有；

开发者可以通过 api 或者 web portal 读写这些数据。

## 2. 玩家资料项的读写权限

一般情况下，DS SDK可以读写所有玩家的所有资料；

玩家基础信息对所有玩家都是公开可读的的，但只有玩家本人可以写；

玩家属性和玩家数据则有三个并列的权限维度：internal，client_writable，client_public

![](/markdown/Player-Profile/254d4c05-c092-40ee-81f4-df72a2587390.png)

- internal：是否为内部数据，内部数据仅Server可以读写
  - true: client不可读（此时另两个维度无意义）
  - false: client可读
- client_writable：拥有数据的玩家是否可写这个数据
  - true：拥有数据的玩家可以写
  - false：拥有数据的玩家只能读
- client_public：是否对其他玩家公开
  - true：其他玩家可读
  - false：其他玩家不可读

以下是各种权限组合情况下各个角色的读写权限：

| Internal | client_writeable | client_public | Server     | Client Owner | Client Other |
| -------- | ---------------- | ------------- | ---------- | ------------ | ------------ |
| true     | /                | /             | Read/Write | Invisible    | Invisible    |
| false    | true             | true          | Read/Write | Read/Write   | ReadOnly     |
| false    | true             | false         | Read/Write | Read/Write   | Invisible    |
| false    | false            | true          | Read/Write | ReadOnly     | ReadOnly     |
| false    | false            | false         | Read/Write | ReadOnly     | Invisible    |

## 3. 玩家基础信息

- 玩家基础信息包括：

  - player id：玩家的TGOS唯一标识

  - display name：玩家的昵称

  - avatar url：玩家的个性化头像url地址

  - language：玩家交互界面使用的语言（如果游戏支持多语言本地化）

    ```c++
    struct PlayerInfo {
        std::string player_id;
        std::string display_name;
        std::string avatar_url;
        std::string language;
    };
    ```

- Client SDK相关接口（详见C++头文件附件）：

  - 获取当前玩家的基础信息：

    ```c++
    /*
    * @brief get base info for current logined player.
    * @param callback result callback for the function
    * @return 
    */
    void GetMyPlayerInfo(HttpCallback<PlayerInfo> callback)
    ```

  - 设置当前玩家的基础信息：

    ```c++
    /*
    * @brief set display name for current logined player.
    * @param callback result callback for the function
    * @return 
    */
    void SetMyPlayerName(const std::string &display_name,
                   HttpCallback<Void> callback);
    /*
    * @brief set avatar url for current logined player.
    * @param callback result callback for the function
    * @return 
    */
    void SetMyPlayerAvatar(const std::string &avatar_url,
                     HttpCallback<Void> callback);
    /*
    * @brief set language for current logined player.
    * @param callback result callback for the function
    * @return 
    */
    void SetMyPlayerLanguage(const std::string &language,
                       HttpCallback<Void> callback);
    ```

  - 获取其他玩家的基础信息：

    ```c++
    /*
    * @brief get base info corresponding to player id
    * @param player_id target player's id
    * @param callback result callback for the function
    * @return 
    */
    void GetPlayerInfo(const std::string &player_id,
                       HttpCallback<PlayerInfo> callback);
    ```

- DS SDK 相关接口（详见C++头文件附件）：
  
  - 获取一个玩家基础信息
  
    ```c++
    // get base info for single player.
    void GetPlayerInfo(const std::string &player_id,
                       HttpCallback<PlayerInfo> callback);
    ```
  
  - 获取多个玩家基础信息
  
    ```c++
    // get base info for multiple players
    void GetPlayerInfo(const std::vector<std::string> &player_ids, HttpCallback<std::map<std::string, QueryPlayerInfoResult>> callback);
    ```

## 4. 玩家属性集合

在使用玩家属性集合之前，开发者需要先再web portal上定制游戏玩家的属性集合模板，并且在游戏运行时，开发者只能读写属性模板中有的属性，而且不能删除属性。**玩家属性可被用于对局匹配规则和排行榜**。

![](/markdown/Player-Profile/015713c4-aa00-44ef-adc1-0d9e1dfe5aff.png)

- Client SDK相关接口（详见C++头文件附件）：
  - 获取当前玩家的全部属性
  
    ```C++
    /**
    * Query all properties of the current player
    * @param callback Result callback of the request
    */
    virtual void GetMyPlayerProperties(HttpCallback<KVDataMap> callback) = 0;
    
    using KVDataMap = std::map<std::string, KVData>;
    struct KVData {
        enum class Type : int { Int = 0, Float = 1, String = 2 };
        KVData(std::string k, int v);
        KVData(std::string k, float v);
        KVData(std::string k, std::string v);
    
        const std::string& StringValue() const { return value; }
        int IntValue() const { return atoi(value.c_str()); }
        float FloatValue() const { return atof(value.c_str()); }
        Type type() const { return type_; }
        bool Empty() const { return key.empty(); }
    
        std::string key;
        std::string value;
        Type type_;
    };
    ```
  
  - 获取当前玩家的部分属性
  
  - 设置当前玩家的指定属性
  
  - 获取一个玩家的所有公开属性
  
  - 获取多个玩家的所有公开属性
- DS SDK相关接口（详见C++头文件附件）：
  - 获取一个玩家的指定的或全部属性
  
    ```C++
    // get properties for single player
    void GetPlayerProperties(const KVDataQuery& query,	                 HttpCallback<KVDataQueryResult> callback);
    
    // Leave the query_keys empty to query all properties 
    // or specified some keys you want to query
    struct KVDataQuery {
        std::string player_id;
        std::vector<std::string> query_keys;
    };
    
    struct KVDataQueryResult : public KVDataOperationResult {
        std::string player_id;
        KVDataList data_list;
    };
    
    using KVDataList = std::vector<KVData>;
    ```
  
  - 获取多个玩家的指定的或全部属性
  
  - 设置一个玩家的指定的属性
  
  - 设置多个玩家的指定的属性

## 5. 玩家数据集合

开发者可以单独为某个玩家自由的赋予玩家数据，不用像玩家属性那样受web portal设置的模板限制。

- Client SDK相关接口（详见C++头文件附件）：
  - 获取当前玩家的全部数据
  
    ```c++
    /**
     * Query all KVData of the current player
     * @param callback Result callback of the request
     */
    virtual void GetMyPlayerKVData(HttpCallback<KVDataMap> callback) = 0;
    ```
  
  - 获取当前玩家的一些指定的数据
  
  - 添加或更新当前玩家的一些指定的数据
  
  - 移除当前玩家的一些指定的数据
  
  - 获取一个玩家的指定的或全部公开数据
  
  - 获取多个玩家的指定的或全部公开数据
- DS SDK相关接口（详见C++头文件附件）：
  - 获取一个玩家的指定的或全部数据
  
    ```c++
    // get data for single player
    void GetPlayerData(const KVDataQuery& query,
                       HttpCallback<KVDataQueryResult> callback);
    ```
  
  - 获取多个玩家的指定的或全部数据
  
  - 设置一个玩家的指定的数据
  
  - 设置多个玩家的指定的数据
  
  - 删除一个玩家的指定的数据
  
  - 删除多个玩家的指定的数据

## 6. 使用说明

- Client SDK 使用说明

  - 获得 IPlayerProfile 对象：

    ```C++
    IPlayerProfile *PlayerProfile();
    ```

  - 使用 IPlayerProfile 相关接口读写数据；

- DS SDK 使用说明

  - 在 tgos_server.h 找到 \*Player\*() 相关接口，直接调用这些全局函数；

## 7. Web portal 相关功能

- 查询玩家

  ![](/markdown/Player-Profile/2668698e-2bf8-45f5-9587-a3fe310c9b83.png)

- 玩家信息

  ![](/markdown/Player-Profile/de3dc379-3a15-4551-9e09-a536b998ec84.png)

- 查看指定玩家的数据

  ![](/markdown/Player-Profile/22ffff42-6777-47de-9c97-f7872324aba3.png)

- 查看指定玩家的属性

  ![](/markdown/Player-Profile/e64a6974-a2e3-4e53-99d7-8d06f9f9e7de.png)

- 查看登录流水

  ![](/markdown/Player-Profile/ce3b76e0-57b5-4ed4-a89a-d9b84704d145.png)

- 定制玩家属性模板

  ![](/markdown/Player-Profile/015713c4-aa00-44ef-adc1-0d9e1dfe5aff.png)