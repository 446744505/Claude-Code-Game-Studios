# Godot 网络 — 速查

上次核对：2026-02-12 | 引擎：Godot 4.6

## 相对约 4.3 以来的变化（LLM 截断知识）

### 4.6 变更
- **破坏性变更中的网络小节**：请参阅官方迁移指南中 4.5→4.6 层级的具体说明

### 4.5 变更
- **无重大网络 API 破坏性变更** — 核心多人 API 保持稳定

## 当前 API 模式

### 高层多人（High-Level Multiplayer）
```gdscript
# 服务端
func host_game(port: int = 9999) -> void:
    var peer := ENetMultiplayerPeer.new()
    peer.create_server(port)
    multiplayer.multiplayer_peer = peer
    multiplayer.peer_connected.connect(_on_peer_connected)
    multiplayer.peer_disconnected.connect(_on_peer_disconnected)

# 客户端
func join_game(address: String, port: int = 9999) -> void:
    var peer := ENetMultiplayerPeer.new()
    peer.create_client(address, port)
    multiplayer.multiplayer_peer = peer
```

### RPC
```gdscript
# 服务端权威模式
@rpc("any_peer", "call_local", "reliable")
func request_action(action_data: Dictionary) -> void:
    if not multiplayer.is_server():
        return
    # 在服务端校验，再广播
    _execute_action.rpc(action_data)

@rpc("authority", "call_local", "reliable")
func _execute_action(action_data: Dictionary) -> void:
    # 所有对等端执行已校验的动作
    pass
```

### MultiplayerSpawner 与 MultiplayerSynchronizer
```gdscript
# 使用 MultiplayerSpawner 做节点自动复制
# 使用 MultiplayerSynchronizer 做属性同步

# MultiplayerSynchronizer 设置：
# 1. 作为待同步节点的子节点添加
# 2. 在编辑器中配置要复制的属性
# 3. 设置可见性过滤器以控制相关性（relevancy）
```

### SceneMultiplayer 配置
```gdscript
func _ready() -> void:
    var scene_mp := multiplayer as SceneMultiplayer
    scene_mp.auth_callback = _authenticate_peer
    scene_mp.server_relay = false  # 对等端直连

func _authenticate_peer(id: int, data: PackedByteArray) -> void:
    # 自定义鉴权逻辑
    pass
```

## 常见错误
- 客户端到服务端的 RPC 未使用 `"any_peer"`（默认仅 authority）
- 未在服务端校验就信任客户端数据
- 对游戏状态变更使用 `"unreliable"`（仅宜用于位置等更新）
- 未对生成（spawn）的节点设置多人权威（`set_multiplayer_authority()`）
