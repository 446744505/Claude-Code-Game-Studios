# Unity 6.3 — 音频模块参考

**最后核对：** 2026-02-13  
**知识缺口：** Unity 6 音频混音器改进

---

## 概述

Unity 6.3 音频系统：
- **AudioSource**：在 GameObject 上播放声音
- **Audio Mixer**：混音、效果处理、动态混音
- **Spatial Audio**：三维定位音效

---

## 基础音频播放

### AudioSource 组件

```csharp
AudioSource audioSource = GetComponent<AudioSource>();

// ✅ 播放
audioSource.Play();

// ✅ 延迟播放
audioSource.PlayDelayed(0.5f); // 0.5 秒

// ✅ 一次性播放（不打断当前声音）
audioSource.PlayOneShot(clip);

// ✅ 停止
audioSource.Stop();

// ✅ 暂停 / 恢复
audioSource.Pause();
audioSource.UnPause();
```

### 在指定位置播放声音（静态方法）

```csharp
// ✅ 快速 3D 声音播放（播放结束后自动销毁）
AudioSource.PlayClipAtPoint(clip, transform.position);

// ✅ 指定音量
AudioSource.PlayClipAtPoint(clip, transform.position, 0.7f);
```

---

## 3D 空间音频

### AudioSource 3D 设置

```csharp
AudioSource source = GetComponent<AudioSource>();

// Spatial Blend：0 = 2D，1 = 3D
source.spatialBlend = 1.0f; // 完全 3D

// 多普勒效应（随速度改变音高）
source.dopplerLevel = 1.0f;

// 距离衰减
source.minDistance = 1f;   // 此距离内为满音量
source.maxDistance = 50f;  // 超过此距离听不见
source.rolloffMode = AudioRolloffMode.Logarithmic; // 自然衰减
```

### 音量衰减曲线
- **Logarithmic（对数）**：自然、写实（推荐）
- **Linear（线性）**：均匀下降
- **Custom（自定义）**：自定义曲线

---

## Audio Mixer（高级混音）

### 创建 Audio Mixer

1. `Assets > Create > Audio Mixer`
2. 打开混音器：`Window > Audio > Audio Mixer`
3. 创建分组：Master > SFX、Music、Dialogue

### 将 AudioSource 指派到 Mixer 分组

```csharp
using UnityEngine.Audio;

public AudioMixerGroup sfxGroup;

void Start() {
    AudioSource source = GetComponent<AudioSource>();
    source.outputAudioMixerGroup = sfxGroup; // 路由到 SFX 分组
}
```

### 从代码控制 Mixer

```csharp
using UnityEngine.Audio;

public AudioMixer audioMixer;

// ✅ 设置音量（暴露参数）
audioMixer.SetFloat("MusicVolume", -10f); // 分贝（-80 到 0）

// ✅ 读取音量
audioMixer.GetFloat("MusicVolume", out float volume);

// 将线性（0–1）转为分贝
float volumeDB = Mathf.Log10(volumeLinear) * 20f;
audioMixer.SetFloat("MusicVolume", volumeDB);
```

### 暴露 Mixer 参数
在 Audio Mixer 窗口中：
1. 右键参数（例如 Volume）
2. 选择「Expose 'Volume' to script」
3. 在「Exposed Parameters」标签中重命名（例如 `MusicVolume`）

---

## 音频效果

### 为 Mixer 分组添加效果

在 Audio Mixer 中：
- 点击分组（例如 SFX）
- 点击「Add Effect」
- 选择：Reverb、Echo、Low Pass、High Pass、Distortion 等

### 对话时压低音乐（侧链）

```csharp
// 在 Audio Mixer 中设置：
// 1. 创建「Duck Volume」快照
// 2. 在该快照中降低音乐音量
// 3. 播放对话时过渡到该快照

public AudioMixerSnapshot normalSnapshot;
public AudioMixerSnapshot duckedSnapshot;

public void PlayDialogue(AudioClip clip) {
    duckedSnapshot.TransitionTo(0.5f); // 0.5 秒过渡
    audioSource.PlayOneShot(clip);
    Invoke(nameof(RestoreMusic), clip.length);
}

void RestoreMusic() {
    normalSnapshot.TransitionTo(1.0f); // 1 秒过渡回原状
}
```

---

## 音频性能

### 优化音频加载

```csharp
// 音频导入设置（Inspector）：
// - Load Type：
//   - Decompress On Load：小片段（SFX），完整载入内存
//   - Compressed In Memory：中等片段，运行时解压（推荐）
//   - Streaming：大文件（音乐），从磁盘流式读取

// Compression Format：
// - PCM：未压缩，质量最高，体积最大
// - ADPCM：约 3.5 倍压缩，适合 SFX（SFX 推荐）
// - Vorbis/MP3：高压缩，适合音乐（音乐推荐）
```

### 预加载音频

```csharp
// 播放前预加载音频片段（避免卡顿）
audioSource.clip.LoadAudioData();

// 检查是否已加载
if (audioSource.clip.loadState == AudioDataLoadState.Loaded) {
    audioSource.Play();
}
```

---

## 音乐系统

### 曲目交叉淡入淡出

```csharp
public IEnumerator CrossfadeMusic(AudioSource from, AudioSource to, float duration) {
    float elapsed = 0f;
    to.Play();

    while (elapsed < duration) {
        elapsed += Time.deltaTime;
        float t = elapsed / duration;

        from.volume = Mathf.Lerp(1f, 0f, t);
        to.volume = Mathf.Lerp(0f, 1f, t);

        yield return null;
    }

    from.Stop();
}
```

### 无缝音乐循环

```csharp
// 音频导入设置：
// - 勾选「Loop」以实现无缝循环
audioSource.loop = true;
```

---

## 常见模式

### 随机音高变化（减轻重复感）

```csharp
void PlaySoundWithVariation(AudioClip clip) {
    AudioSource source = GetComponent<AudioSource>();
    source.pitch = Random.Range(0.9f, 1.1f); // 约 ±10% 音高变化
    source.PlayOneShot(clip);
}
```

### 脚步声（从数组随机）

```csharp
public AudioClip[] footstepClips;

void PlayFootstep() {
    AudioClip clip = footstepClips[Random.Range(0, footstepClips.Length)];
    AudioSource.PlayClipAtPoint(clip, transform.position, 0.5f);
}
```

### 判断是否正在播放

```csharp
if (audioSource.isPlaying) {
    // 当前正在播放
}
```

---

## Audio Listener

### 单一 Listener 规则
- 同一时刻只应有一个活动的 `AudioListener`
- 通常挂在主摄像机上

```csharp
// 关闭多余的 Listener
AudioListener listener = GetComponent<AudioListener>();
listener.enabled = false;
```

---

## 调试

### Audio 窗口
- `Window > Audio > Audio Mixer`
- 可视化电平、测试快照

### Audio 设置
- `Edit > Project Settings > Audio`
- 全局音量、DSP 缓冲区大小、扬声器模式

---

## 来源
- https://docs.unity3d.com/6000.0/Documentation/Manual/Audio.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/AudioMixer.html
