## Why

在 SSL/TLS 证书加载过程中，`SSL_CTX_use_enc_certificate()` 和 `SSL_CTX_use_sign_certificate()` 两个函数对证书的 Key Usage (KU) 扩展进行了严格验证。这导致一些 KU 位不完整但实际合法可用的证书被拒绝加载，影响了与旧版 CA 签发证书或非标准证书的兼容性。需要禁用这些检查以提高证书兼容性。

## What Changes

- 在 `ssl/ssl_rsa.c` 的 `SSL_CTX_use_enc_certificate()` 函数中，注释掉对 `X509v3_KU_KEY_ENCIPHERMENT` 和 `X509v3_KU_DATA_ENCIPHERMENT` 位的检查逻辑
- 在 `ssl/ssl_rsa.c` 的 `SSL_CTX_use_sign_certificate()` 函数中，注释掉对 `X509v3_KU_DIGITAL_SIGNATURE`、`X509v3_KU_KEY_CERT_SIGN` 和 `X509v3_KU_CRL_SIGN` 位的检查逻辑

## Capabilities

### New Capabilities

- `disable-ku-check`: 禁用 SSL 证书加载函数中的 Key Usage 扩展验证，允许 KU 位不完整的证书被成功加载

### Modified Capabilities

（无现有 capability 需要修改）

## Impact

- **ssl/ssl_rsa.c**: 修改 `SSL_CTX_use_enc_certificate()` 和 `SSL_CTX_use_sign_certificate()` 两个函数，注释掉 KU 检查代码块
- **安全性**: 移除 KU 验证后，不符合密钥用途要求的证书也可以被加载，可能存在误用风险，但该检查本身不属于 TLS 协议握手的强制要求，而是 Tongsuo 的额外限制
- **兼容性**: 提高了对各类证书的兼容性，特别是旧版 CA 签发的 KU 位缺失或不完整的证书
