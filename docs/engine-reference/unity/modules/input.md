# Unity 6.3 — Input 模块参考

**最后核对：** 2026-02-13  
**知识缺口：** Unity 6 使用新版 Input System（旧版 Input 已弃用）

---

## 概述

Unity 6 输入体系：
- **Input System 包**（推荐）：跨平台、可重绑、现代方案  
- **旧版 Input Manager**：已弃用，新项目应避免使用  

---

## 相对 2022 LTS 的主要变化

### Unity 6 中旧版 Input 已弃用

```csharp
// ❌ 已弃用：Input 类
if (Input.GetKeyDown(KeyCode.Space)) { }

// ✅ 新版：Input System 包
using UnityEngine.InputSystem;
if (Keyboard.current.spaceKey.wasPressedThisFrame) { }
```

**需要迁移：** 安装 `com.unity.inputsystem` 包。

---

## Input System 包配置

### 安装
1. `窗口 > 包管理器`（Window > Package Manager）
2. 搜索 “Input System”
3. 安装包
4. 按提示重启 Unity

### 启用新 Input System  
`编辑 > 项目设置 > 玩家 > Active Input Handling`（Edit > Project Settings > Player）：
- **Input System Package (New)** ✅ 推荐  
- **Both**（迁移过渡期可用）

---

## Input Actions（推荐用法）

### 创建 Input Actions 资源

1. `资源 > 创建 > Input Actions`（Assets > Create > Input Actions）
2. 命名（例如 “PlayerControls”）
3. 打开资源，定义动作：

```
Action Maps:
  Gameplay
    Actions:
      - Move (Value, Vector2)
      - Jump (Button)
      - Fire (Button)
      - Look (Value, Vector2)
```

4. **生成 C# 类**：在 Inspector 中勾选 “Generate C# Class”
5. 点击 “Apply”

### 使用生成的输入类

```csharp
using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerController : MonoBehaviour {
    private PlayerControls controls;

    void Awake() {
        controls = new PlayerControls();

        // 订阅动作
        controls.Gameplay.Jump.performed += ctx => Jump();
        controls.Gameplay.Fire.performed += ctx => Fire();
    }

    void OnEnable() => controls.Enable();
    void OnDisable() => controls.Disable();

    void Update() {
        // 读取持续输入
        Vector2 move = controls.Gameplay.Move.ReadValue<Vector2>();
        transform.Translate(new Vector3(move.x, 0, move.y) * Time.deltaTime);

        Vector2 look = controls.Gameplay.Look.ReadValue<Vector2>();
        // 应用相机旋转
    }

    void Jump() {
        Debug.Log("Jump!");
    }

    void Fire() {
        Debug.Log("Fire!");
    }
}
```

---

## 直连设备访问（快速但不推荐长期依赖）

### 键盘

```csharp
using UnityEngine.InputSystem;

void Update() {
    // 当前状态
    if (Keyboard.current.spaceKey.isPressed) { }

    // 本帧刚按下
    if (Keyboard.current.spaceKey.wasPressedThisFrame) { }

    // 本帧刚释放
    if (Keyboard.current.spaceKey.wasReleasedThisFrame) { }
}
```

### 鼠标

```csharp
using UnityEngine.InputSystem;

void Update() {
    // 鼠标位置
    Vector2 mousePos = Mouse.current.position.ReadValue();

    // 鼠标增量（移动）
    Vector2 mouseDelta = Mouse.current.delta.ReadValue();

    // 鼠标按键
    if (Mouse.current.leftButton.wasPressedThisFrame) { }
    if (Mouse.current.rightButton.isPressed) { }

    // 滚轮
    Vector2 scroll = Mouse.current.scroll.ReadValue();
}
```

### 手柄

```csharp
using UnityEngine.InputSystem;

void Update() {
    Gamepad gamepad = Gamepad.current;
    if (gamepad == null) return; // 未连接手柄

    // 按键
    if (gamepad.buttonSouth.wasPressedThisFrame) { } // A / Cross
    if (gamepad.buttonWest.wasPressedThisFrame) { }  // X / Square

    // 摇杆
    Vector2 leftStick = gamepad.leftStick.ReadValue();
    Vector2 rightStick = gamepad.rightStick.ReadValue();

    // 扳机
    float leftTrigger = gamepad.leftTrigger.ReadValue();
    float rightTrigger = gamepad.rightTrigger.ReadValue();

    // 十字键
    Vector2 dpad = gamepad.dpad.ReadValue();
}
```

### 触摸（移动端）

```csharp
using UnityEngine.InputSystem;
using UnityEngine.InputSystem.EnhancedTouch;

void OnEnable() {
    EnhancedTouchSupport.Enable();
}

void Update() {
    foreach (var touch in UnityEngine.InputSystem.EnhancedTouch.Touch.activeTouches) {
        Debug.Log($"Touch at {touch.screenPosition}");
    }
}
```

---

## Input Action 回调

### 动作回调（事件驱动）

```csharp
// started：输入开始（例如扳机轻微按下）
controls.Gameplay.Fire.started += ctx => Debug.Log("Fire started");

// performed：动作已触发（例如按键完全按下）
controls.Gameplay.Fire.performed += ctx => Debug.Log("Fire performed");

// canceled：输入释放或被打断
controls.Gameplay.Fire.canceled += ctx => Debug.Log("Fire canceled");
```

### 上下文数据

```csharp
controls.Gameplay.Move.performed += ctx => {
    Vector2 value = ctx.ReadValue<Vector2>();
    float duration = ctx.duration; // 按住时长
    InputControl control = ctx.control; // 触发该事件的设备/控件
};
```

---

## Control Scheme 与设备切换

### 在 Input Actions 资源中定义 Control Scheme

```
Control Schemes:
  - Keyboard&Mouse (Keyboard, Mouse)
  - Gamepad (Gamepad)
  - Touch (Touchscreen)
```

### 设备变化时自动切换

```csharp
controls.Gameplay.Move.performed += ctx => {
    if (ctx.control.device is Keyboard) {
        Debug.Log("Using keyboard");
    } else if (ctx.control.device is Gamepad) {
        Debug.Log("Using gamepad");
    }
};
```

---

## 重绑（运行时改键）

### 交互式重绑

```csharp
using UnityEngine.InputSystem;

public void RebindJumpKey() {
    var rebindOperation = controls.Gameplay.Jump.PerformInteractiveRebinding()
        .WithControlsExcluding("Mouse") // 排除鼠标绑定
        .OnComplete(operation => {
            Debug.Log("Rebind complete");
            operation.Dispose();
        })
        .Start();
}
```

### 保存 / 加载绑定

```csharp
// 保存
string rebinds = controls.SaveBindingOverridesAsJson();
PlayerPrefs.SetString("InputBindings", rebinds);

// 加载
string rebinds = PlayerPrefs.GetString("InputBindings");
controls.LoadBindingOverridesFromJson(rebinds);
```

---

## 动作类型（Action Types）

### Button（按下/释放）
- 单次按下与释放  
- 示例：跳跃、开火  

### Value（连续值）
- 连续数值（float、Vector2）  
- 示例：移动、视角、瞄准  

### Pass-Through（直通）
- 不做处理，直接传递数值  
- 示例：鼠标位置  

---

## Processors（输入修饰器）

### Scale

```csharp
// 在 Input Actions 资源中：Action > Properties > Processors > Add > Scale
// 将输入乘以系数（例如反转 Y 轴）
```

### Invert

```csharp
// 在 Input Actions 资源中：Action > Properties > Processors > Add > Invert
// 翻转输入符号
```

### Dead Zone

```csharp
// 在 Input Actions 资源中：Action > Properties > Processors > Add > Stick Deadzone
// 忽略摇杆小幅度偏移
```

---

## PlayerInput 组件（简化配置）

### 自动输入设置

```csharp
// 添加组件：Player Input
// 指定 Input Actions 资源
// 行为：Send Messages / Invoke Unity Events / Invoke C# Events

// Send Messages 示例：
public class Player : MonoBehaviour {
    public void OnMove(InputValue value) {
        Vector2 move = value.Get<Vector2>();
        // 处理移动
    }

    public void OnJump(InputValue value) {
        if (value.isPressed) {
            Jump();
        }
    }
}
```

---

## 调试

### Input Debugger
- `窗口 > 分析 > Input Debugger`（Window > Analysis > Input Debugger）
- 查看活动设备、输入数值、动作状态  

---

## 来源
- https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/index.html
- https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/QuickStartGuide.html
