## 1. 禁用加密证书 KU 检查

- [x] 1.1 在 `ssl/ssl_rsa.c` 的 `SSL_CTX_use_enc_certificate()` 函数中，使用 C 块注释 `/* do not check key usage ... */` 包裹对 `X509v3_KU_KEY_ENCIPHERMENT` 和 `X509v3_KU_DATA_ENCIPHERMENT` 的 KU 检查代码块（约第 1443-1447 行）

## 2. 禁用签名证书 KU 检查

- [x] 2.1 在 `ssl/ssl_rsa.c` 的 `SSL_CTX_use_sign_certificate()` 函数中，使用 C 块注释 `/* do not check key usage ... */` 包裹对 `X509v3_KU_DIGITAL_SIGNATURE`、`X509v3_KU_KEY_CERT_SIGN` 和 `X509v3_KU_CRL_SIGN` 的 KU 检查代码块（约第 1525-1530 行）

## 3. 验证与测试（须在 `build` 目录中进行 out-of-source 构建，不得污染源码目录）

- [x] 3.1 创建 `build` 目录，在其中执行 `../Configure` 并 `make` 确认编译通过
- [x] 3.2 验证 `SSL_CTX_use_enc_certificate()` 可以成功加载不含 `KEY_ENCIPHERMENT` KU 位的加密证书
- [x] 3.3 验证 `SSL_CTX_use_sign_certificate()` 可以成功加载不含 `DIGITAL_SIGNATURE` KU 位的签名证书
