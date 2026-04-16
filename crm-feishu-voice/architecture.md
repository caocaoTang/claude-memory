---
name: crm-feishu-voice 架构全貌
description: 飞书语音录制+实时转写应用，Java 17 + Vue 3，阿里云听悟 ASR，WebSocket 双向通信
type: project
---

## 项目概述
飞书语音录制与实时转写平台（crm-feishu-voice），支持纯录音和实时 ASR 两种模式，嵌入飞书生态。

## 技术栈
- **后端**: Java 17, Spring Boot (youzan-boot-parent 2.3.0), Dubbo 3.2.6, MyBatis, OkHttp3
- **前端**: Vue 3 + TypeScript + Pinia + Element Plus + Vite
- **外部服务**: 阿里云听悟(ASR)、飞书 JSSDK、腾讯云声纹(可选)、七牛云存储(可选)
- **端口**: HTTP 7002, Dubbo 7102, Metadata 7103

## Maven 多模块
- `crm-feishu-voice-web`: Spring Boot 入口 + REST 控制器 + WebSocket Handler
- `crm-feishu-voice-biz`: 业务服务层 (ASR/Session/Storage/SpeakerMapping)
- `crm-feishu-voice-config`: 配置属性 Bean（Tingwu/WebSocket/Qiniu/FeishuJSSdk）
- `crm-feishu-voice-dependency`: 外部 API 客户端（FeishuOpenApi/TencentVoiceprint/Qiniu）
- `crm-feishu-voice-domain`: DTO/枚举/请求响应
- `crm-feishu-voice-dal`: 实体 PO + MyBatis Mapper（已建模但未启用数据库）
- `crm-feishu-voice-frontend`: Vue 3 前端

## 核心数据流

### 纯录音模式
前端录音 → stop 后 Blob 上传 → POST /api/v1/recording/upload → 本地文件存储

### 实时 ASR 模式
1. 前端建 WebSocket 连接 /ws → send init 消息
2. 后端调阿里云听悟 CreateTask 获取 MeetingJoinUrl
3. 后端建 WebSocket 连阿里云听悟，发 StartTranscription
4. 前端每 300ms 发 PCM 音频帧（16kHz 16bit）→ 后端转发给听悟
5. 听悟返回 TranscriptionResultChanged / SentenceEnd → 后端封装推给前端
6. 前端实时渲染转写文本 + 说话人标识

## API 端点
- `GET  /api/v1/session/list?userId=...` — 会话列表
- `GET  /api/v1/session/{sessionId}` — 会话详情
- `PUT  /api/v1/session/{sessionId}/end` — 结束会话
- `POST /api/v1/recording/upload` — 上传录音
- `GET  /api/v1/recording/{sessionId}` — 录音元数据
- `GET  /api/v1/transcription/{sessionId}` — 转写文本
- `GET  /api/v1/feishu/jssdk/config?url=...` — JSSDK 签名
- `WS   /ws` — 实时流

## WebSocket 协议
- Client→Server: init(json), audio(binary PCM), ping(json), end(json)
- Server→Client: init_success, result(转写), speaker_update, pong, error

## 前端架构
- Composables 模式: useAudioCapture (抽象层) → useFeishuRecorder(移动端) / useWebAudioRecorder(PC/浏览器)
- 平台检测: 飞书移动(JSSDK RecorderManager + 3秒超时降级) / 飞书PC(WebAudio) / 浏览器(WebAudio)
- Pinia store 管理录音会话、实时对话列表

## 数据库建模（ConcurrentHashMap 暂存，未接库）
- FeishuVoiceSessionPO: 会话 (sessionId, userId, mode, status, platform, duration, recordingUrl)
- RecordingFilePO: 录音文件 (sessionId, fileUrl, size, format, duration)
- TranscriptionSegmentPO: 转写片段 (sessionId, dialogueId, speakerId, speakerName, text, timestamps, isFinal)

## 当前状态
- 数据库持久化未启用，用内存 Map
- 文件存储用本地 ~/crm-feishu-voice/recordings/
- 腾讯声纹客户端已集成但未使用，说话人名称自动分配
- SessionListView/TranscriptionView 为占位组件

**Why:** 了解项目全貌便于后续开发维护
**How to apply:** 涉及此项目的任何修改或问题排查，基于此架构理解进行
