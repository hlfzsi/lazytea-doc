# 请求/响应 通信协议

## 适用范围

适用于满足以下所有条件的场景：

1. 一级组件与二级组件的交互过程。
2. 发送请求/响应

## 消息结构规范

LazyTea 使用结构化的 JSON 消息格式，以 ASCII 记录分隔符 (`\x1e`) 作为消息边界标识。

基础消息包含以下字段：

- `version`: 协议版本号 (当前为 "1.0")
- `header`: 消息头部信息 (MessageHeader)
- `payload`: 消息载荷数据

消息头部 (MessageHeader) 结构：

- `msg_id`: 唯一消息标识符
- `msg_type`: 消息类型 ("request", "response", "broadcast", "heartbeat")
- `correlation_id`: 关联请求ID（用于响应匹配）
- `timestamp`: 时间戳

## 支持的消息载荷类型

| 载荷类型 | 用途 | 结构 |
|---------|------|------|
| `RequestPayload` | 方法调用请求 | `{method: str, params: Dict[str, Any]}` |
| `ResponsePayload` | 请求响应结果 | `{code: int, time: int, data: Any, error: Optional[str]}` |
| `HeartbeatPayload` | 连接心跳监测 | `{status: str}` |