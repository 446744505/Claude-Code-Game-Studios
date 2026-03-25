# Unreal Engine 5.7 — 音频模块参考

**最后核对：** 2026-02-13  
**知识缺口：** UE 5.7 MetaSounds 生产就绪度

---

## 概览

UE 5.7 音频系统：
- **MetaSounds**：基于节点的程序化音频（推荐，生产就绪）
- **Sound Cues**：旧版基于节点的音频（简单场景可用）
- **Audio Component**：在 Actor 上播放声音

---

## 基础音频播放

### 在位置播放声音

```cpp
#include "Kismet/GameplayStatics.h"

// ✅ 播放 2D 声音（无空间化）
UGameplayStatics::PlaySound2D(GetWorld(), ExplosionSound);

// ✅ 在位置播放声音（3D 空间音频）
UGameplayStatics::PlaySoundAtLocation(GetWorld(), ExplosionSound, GetActorLocation());

// ✅ 指定音量与音高
UGameplayStatics::PlaySoundAtLocation(GetWorld(), ExplosionSound, GetActorLocation(), 0.7f, 1.2f);
```

---

## Audio Component

### Audio Component（持续播放的声音）

```cpp
// 创建音频组件
UAudioComponent* AudioComp = CreateDefaultSubobject<UAudioComponent>(TEXT("Audio"));
AudioComp->SetupAttachment(RootComponent);
AudioComp->SetSound(LoopingAmbience);

// 播放 / 停止
AudioComp->Play();
AudioComp->Stop();

// 淡入 / 淡出
AudioComp->FadeIn(2.0f); // 2 秒
AudioComp->FadeOut(1.5f, 0.0f); // 1.5 秒淡到音量 0

// 调节音量 / 音高
AudioComp->SetVolumeMultiplier(0.5f);
AudioComp->SetPitchMultiplier(1.2f);
```

---

## 3D 空间音频

### 衰减设置

```cpp
// 创建 Sound Attenuation 资源：
// 内容浏览器 > Sounds > Sound Attenuation

// 配置项：
// - Attenuation Shape：Sphere、Capsule、Box、Cone
// - Falloff Distance：声音听不清的距离
// - Attenuation Function：Linear、Logarithmic、Inverse 等

// 在 C++ 中指定：
AudioComp->AttenuationSettings = AttenuationAsset;
```

### 在代码中覆盖衰减

```cpp
FSoundAttenuationSettings AttenuationOverride;
AttenuationOverride.AttenuationShape = EAttenuationShape::Sphere;
AttenuationOverride.FalloffDistance = 1000.0f;
AttenuationOverride.AttenuationShapeExtents = FVector(1000.0f);

AudioComp->AttenuationOverrides = AttenuationOverride;
AudioComp->bOverrideAttenuation = true;
```

---

## MetaSounds（程序化音频）

### 创建 MetaSound Source

1. 内容浏览器 > Sounds > MetaSound Source  
2. 打开 MetaSound 编辑器  
3. 搭建节点图：  
   - **Inputs**：触发器、参数  
   - **Generators**：振荡器、噪声、采样  
   - **Modulators**：包络、LFO  
   - **Effects**：滤波、混响、延迟  
   - **Output**：音频输出  

### 播放 MetaSound

```cpp
// 与普通声音一样播放
UGameplayStatics::PlaySound2D(GetWorld(), MetaSoundSource);

// 或使用 Audio Component
AudioComp->SetSound(MetaSoundSource);
AudioComp->Play();
```

### 设置 MetaSound 参数

```cpp
// 在 MetaSound 中定义参数（带暴露参数的 Input 节点）
// 在 C++ 中设置：
AudioComp->SetFloatParameter(FName("Volume"), 0.8f);
AudioComp->SetIntParameter(FName("OctaveShift"), 2);
AudioComp->SetBoolParameter(FName("EnableReverb"), true);
```

---

## Sound Cues（旧版）

### 创建 Sound Cue

1. 内容浏览器 > Sounds > Sound Cue  
2. 打开 Sound Cue 编辑器  
3. 添加节点：Random、Modulator、Mixer 等  

### 使用 Sound Cue

```cpp
// 与普通声音一样播放
UGameplayStatics::PlaySound2D(GetWorld(), SoundCue);
```

---

## Sound Class 与 Sound Mix

### Sound Class（音量分组）

```cpp
// 创建 Sound Class：内容浏览器 > Sounds > Sound Class
// 层级示例：Master > Music、SFX、Dialogue

// 赋给声音资源：
// Sound Wave > Sound Class = SFX

// 在 C++ 中设置音量：
UAudioSettings* AudioSettings = GetMutableDefault<UAudioSettings>();
// 通过 Sound Class 层级配置
```

### Sound Mix（动态混音）

```cpp
// 创建 Sound Mix 资源
// 定义调整项：例如对话时压低音乐等

// 推入 sound mix
UGameplayStatics::PushSoundMixModifier(GetWorld(), DuckedMusicMix);

// 弹出 sound mix
UGameplayStatics::PopSoundMixModifier(GetWorld(), DuckedMusicMix);
```

---

## 音频遮挡与混响

### 音频遮挡（墙体阻挡声音）

```cpp
// 在 Audio Component 上启用：
AudioComp->bEnableOcclusion = true;

// 需要带碰撞的几何体
```

### Reverb Volume

```cpp
// 在关卡中添加 Audio Volume（Volumes > Audio Volume）
// 在细节面板中配置混响设置
// 进入 Volume 后，Audio Component 会自动拾取混响
```

---

## 常见模式

### 脚步声（随机变体）

```cpp
// 使用带 Random 节点的 Sound Cue，或：
UPROPERTY(EditAnywhere, Category = "Audio")
TArray<TObjectPtr<USoundBase>> FootstepSounds;

void PlayFootstep() {
    int32 Index = FMath::RandRange(0, FootstepSounds.Num() - 1);
    UGameplayStatics::PlaySoundAtLocation(GetWorld(), FootstepSounds[Index], GetActorLocation());
}
```

### 音乐交叉淡入淡出

```cpp
UAudioComponent* MusicA;
UAudioComponent* MusicB;

void CrossfadeMusic(float Duration) {
    MusicA->FadeOut(Duration, 0.0f);
    MusicB->FadeIn(Duration);
}
```

### 判断声音是否在播放

```cpp
if (AudioComp->IsPlaying()) {
    // 正在播放
}
```

---

## 音频并发（Concurrency）

### 限制同时播放实例数

```cpp
// 创建 Sound Concurrency 资源：
// 内容浏览器 > Sounds > Sound Concurrency

// 配置：
// - Max Count：该声音的最大实例数
// - Resolution Rule：Stop Oldest、Stop Quietest 等

// 赋给声音：
// Sound Wave > Concurrency Settings
```

---

## 性能提示

### 音频优化

```cpp
// 压缩设置（Sound Wave 资源）：
// - Compression Quality：40（质量/体积平衡）
// - Streaming：大文件（如音乐）可开启

// 降低混音开销：
// - 通过 Sound Concurrency 限制并发声音数
// - 使用简单的衰减形状

// 对远处 Actor 关闭音频：
if (Distance > MaxAudibleDistance) {
    AudioComp->Stop();
}
```

---

## 调试

### 音频调试命令

```cpp
// 控制台命令：
// au.Debug.Sounds 1 — 显示活动中的声音
// au.3dVisualize.Enabled 1 — 可视化 3D 音频
// stat soundwaves — 显示声音统计
// stat soundmixes — 显示活动中的 sound mix
```

---

## 来源
- https://docs.unrealengine.com/5.7/en-US/audio-in-unreal-engine/
- https://docs.unrealengine.com/5.7/en-US/metasounds-in-unreal-engine/
