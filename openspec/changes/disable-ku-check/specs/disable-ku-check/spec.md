## ADDED Requirements

### Requirement: 加密证书加载时跳过 Key Usage 检查

`SSL_CTX_use_enc_certificate()` 函数 SHALL 在加载加密证书时不检查证书的 Key Usage 扩展。即使证书不具有 `KEY_ENCIPHERMENT` 或 `DATA_ENCIPHERMENT` KU 位，函数 SHALL 继续执行后续逻辑而非返回错误。

#### Scenario: 加载无 KU 扩展的加密证书

- **WHEN** 使用 `SSL_CTX_use_enc_certificate()` 加载一个不含 Key Usage 扩展的 SM2 加密证书
- **THEN** 函数 SHALL 返回成功（返回值 > 0），证书被正确设置到 SSL_CTX 中

#### Scenario: 加载 KU 位不含 KEY_ENCIPHERMENT 的加密证书

- **WHEN** 使用 `SSL_CTX_use_enc_certificate()` 加载一个 Key Usage 仅包含 `DIGITAL_SIGNATURE` 位的 SM2 证书
- **THEN** 函数 SHALL 返回成功（返回值 > 0），而非因 KU 不匹配返回错误

#### Scenario: 加载正常 KU 的加密证书仍然成功

- **WHEN** 使用 `SSL_CTX_use_enc_certificate()` 加载一个 Key Usage 包含 `KEY_ENCIPHERMENT` 位的 SM2 加密证书
- **THEN** 函数 SHALL 返回成功（返回值 > 0）

### Requirement: 签名证书加载时跳过 Key Usage 检查

`SSL_CTX_use_sign_certificate()` 函数 SHALL 在加载签名证书时不检查证书的 Key Usage 扩展。即使证书不具有 `DIGITAL_SIGNATURE`、`KEY_CERT_SIGN` 或 `CRL_SIGN` KU 位，函数 SHALL 继续执行后续逻辑而非返回错误。

#### Scenario: 加载无 KU 扩展的签名证书

- **WHEN** 使用 `SSL_CTX_use_sign_certificate()` 加载一个不含 Key Usage 扩展的 SM2 签名证书
- **THEN** 函数 SHALL 返回成功（返回值 > 0），证书被正确设置到 SSL_CTX 中

#### Scenario: 加载 KU 位不含 DIGITAL_SIGNATURE 的签名证书

- **WHEN** 使用 `SSL_CTX_use_sign_certificate()` 加载一个 Key Usage 仅包含 `KEY_ENCIPHERMENT` 位的 SM2 证书
- **THEN** 函数 SHALL 返回成功（返回值 > 0），而非因 KU 不匹配返回错误

#### Scenario: 加载正常 KU 的签名证书仍然成功

- **WHEN** 使用 `SSL_CTX_use_sign_certificate()` 加载一个 Key Usage 包含 `DIGITAL_SIGNATURE` 位的 SM2 签名证书
- **THEN** 函数 SHALL 返回成功（返回值 > 0）

### Requirement: 其他验证逻辑保持不变

禁用 KU 检查后，`SSL_CTX_use_enc_certificate()` 和 `SSL_CTX_use_sign_certificate()` 的其他验证逻辑 SHALL 保持不变，包括：空指针检查、安全等级检查（`ssl_security_cert`）、公钥类型检查（SM2/RSA 判断）。

#### Scenario: 空证书仍然被拒绝

- **WHEN** 使用 `SSL_CTX_use_enc_certificate()` 传入 NULL 指针
- **THEN** 函数 SHALL 返回 0 并报告 `ERR_R_PASSED_NULL_PARAMETER` 错误

#### Scenario: 不支持的公钥类型仍然被拒绝

- **WHEN** 使用 `SSL_CTX_use_enc_certificate()` 加载一个公钥类型为 Ed25519 的证书
- **THEN** 函数 SHALL 返回 0 并报告 `SSL_R_WRONG_CERTIFICATE_TYPE` 错误
