
## 协议地址
  通讯协议基于websocket的文本消息，序列化使用JSON进行编解码。
  * 测试环境ws endpoint：`ws://test-opsconf.payshop.vip/ws/subquery`
  * 正式环境ws endpoint：`ws://opsconf.payshop.vip/ws/subquery`

## 心跳
#### 请求
``` 
{"bizServerIds":["kaibo","operation"],"command":"HeartbeatRequest","timestamp":1586228521252}
``` 
* bizServerIds：需要订阅的业务服务ID
* command：协议指令，共有`HeartbeatRequest`|`HeartbeatResponse`| `RenewRequest`| `RenewResponse` 四个指令
* timestamp：本地毫秒时间戳


#### 响应
```
{"command":"HeartbeatResponse","timestamp":1586228526212}
```
* command：协议指令
* timestamp：服务端毫秒时间戳
<br/>

## 配置更新
#### 请求
```
{"bizServerIds":["kaibo","operation"],"command":"RenewRequest","opsConfItems":[{"bizServerId":"kaibo","itemKey":"kaibo_1","md5Sign":"58333b5d86083e5d89413c4ae91ee7af","version":4},{"bizServerId":"kaibo","itemKey":"itemkey1","md5Sign":"fdd29d4757b6657c8e62850e77fedeee","version":5}]}
```
* bizServerIds：需要订阅的业务服务ID
* command：协议指令
* timestamp：本地毫秒时间戳
* opsConfItems：本地所有的KV配置信息（便于subquery服务进行差异更新下发，为空时表示全量更新）
	* bizServerId：KV配置所属的bizServerId
	* itemKey：KV配置中的KEY
	* version ：本地的版本
	* md5Sign：KV配置中Value的md5签名，可以忽略

#### 响应
```
{"command":"RenewResponse","opsConfItems":[{"bizServerId":"kaibo","itemData":"valuetest","itemKey":"itemkey1","md5Sign":"fdd29d4757b6657c8e62850e77fedeee","version":5}]}
```
* command：协议指令
* opsConfItems：新的KV配置信息
	* bizServerId：KV配置所属的bizServerId
	* itemKey：KV配置中的KEY
 	* itemData：KV配置中的Value
	* version ：服务端版本（`当version为-1时表示删除本条配置`）
	* md5Sign：KV配置中Value的md5签名

消息场景：
* 客户端发起 `RenewRequest`指令请求，Subquery进行响应
* Subquery主动推送配置变更消息
