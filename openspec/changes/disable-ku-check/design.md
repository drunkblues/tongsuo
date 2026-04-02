## Context

Tongsuo 是基于 OpenSSL 的密码学库分支，支持 NTLS（国密 TLS）双证书体系。在 NTLS 中，服务端需要分别加载签名证书和加密证书，对应的 API 为 `SSL_CTX_use_sign_certificate()` 和 `SSL_CTX_use_enc_certificate()`（定义在 `ssl/ssl_rsa.c` 中）。

当前实现中，这两个函数在加载证书时会检查 X.509 证书的 Key Usage (KU) 扩展：
- `SSL_CTX_use_enc_certificate()` 要求证书具有 `KEY_ENCIPHERMENT` 或 `DATA_ENCIPHERMENT` 位
- `SSL_CTX_use_sign_certificate()` 要求证书具有 `DIGITAL_SIGNATURE`、`KEY_CERT_SIGN` 或 `CRL_SIGN` 位

然而，实际部署中存在许多由旧版 CA 或非标准流程签发的证书，其 KU 扩展字段缺失或不完整。这些证书在 TLS 握手中功能正常，但由于上述检查被拒绝加载，造成兼容性问题。

## Goals / Non-Goals

**Goals:**
- 禁用 `SSL_CTX_use_enc_certificate()` 中对加密相关 KU 位的强制检查
- 禁用 `SSL_CTX_use_sign_certificate()` 中对签名相关 KU 位的强制检查
- 保持函数的其他逻辑（空指针检查、安全检查、公钥类型检查）不变

**Non-Goals:**
- 不移除或修改 TLS 握手过程中的 KU 验证逻辑
- 不修改 `SSL_use_certificate()` 等其他证书加载函数
- 不提供运行时开关来动态启用/禁用 KU 检查
- 不修改 X.509 证书验证链中的 KU 检查

## Decisions

### 1. 使用注释方式禁用而非删除代码

**选择**: 使用 C 块注释 `/* ... */` 包裹 KU 检查代码块  
**理由**: 保留原始代码作为文档记录，便于未来需要时快速恢复。这是一个兼容性权衡，而非逻辑错误修复，保留代码能够清楚地表明这是一个有意的决策。  
**备选**: 直接删除代码 — 被否决，因为失去了对原始设计意图的可追溯性。  
**备选**: 使用预处理器宏条件编译 — 被否决，对于仅涉及两处的简单修改，引入编译选项过于复杂。

## Risks / Trade-offs

- **[安全降级]** 移除 KU 检查后，加密证书可能被误用于签名场景，或签名证书被误用于加密场景 → 实际风险较低，因为后续的公钥类型检查（SM2/RSA 判断）和 TLS 握手协议本身会约束证书的使用方式
- **[标准合规性]** RFC 5280 建议验证 Key Usage → 本次修改仅影响证书加载阶段，不影响证书链验证；且 TLS 实现对 KU 的强制检查程度在业界并不统一
