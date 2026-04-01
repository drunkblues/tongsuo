## Why

Tongsuo 在解析使用 pbeWithSHA1And40BitRC2-CBC（OID 1.2.840.113549.1.12.1.6）加密的 PKCS#12 文件时报错，原因是 RC2 底层实现（crypto/rc2/）完全缺失，且 RC2 被默认禁用。为了保证与旧版系统和证书格式的兼容性，需要将 RC2 算法作为 Legacy Provider 的一部分引入。

## What Changes

- 新增 `crypto/rc2/` 底层实现（从 OpenSSL 移植：`rc2_skey.c`、`rc2_ecb.c`、`rc2_enc.c`、`rc2cfb64.c`、`rc2ofb64.c`、`rc2_local.h`）
- 新增 `include/openssl/rc2.h` 公共头文件
- 新增 Provider 层实现：`providers/implementations/ciphers/cipher_rc2.c`、`cipher_rc2.h`、`cipher_rc2_hw.c`
- 新增 RC2 相关 OID（rc2-cbc、rc2-40-cbc、rc2-64-cbc 等）和 PBE OID（pbeWithSHA1And128BitRC2-CBC、pbeWithSHA1And40BitRC2-CBC）
- 在 `crypto/evp/evp_pbe.c` 中注册 RC2 PBE 算法
- 在 `providers/legacyprov.c` 中注册 RC2 cipher 算法
- 更新构建系统（`Configure`、`crypto/build.info`、`providers/implementations/ciphers/build.info`）
- 在 `Configure` 中将 rc2 加入 `@disablables` 并取消默认禁用

## Capabilities

### New Capabilities
- `rc2-cipher`: RC2 对称加密算法的底层实现与 Provider 层封装，支持 CBC、ECB、OFB、CFB 模式及 40/64/128 位有效密钥长度
- `rc2-pbe`: 基于 RC2 的 PKCS#12 PBE 算法支持（pbeWithSHA1And128BitRC2-CBC 和 pbeWithSHA1And40BitRC2-CBC），用于解析旧版 PKCS#12 文件

### Modified Capabilities

（无现有 capability 需要修改）

## Impact

- **构建系统**：`Configure`、`crypto/build.info`、`providers/implementations/ciphers/build.info` 需更新以包含 RC2 编译单元
- **OID 数据库**：`crypto/objects/objects.txt`、`crypto/objects/obj_mac.num`，以及重新生成的 `include/openssl/obj_mac.h`、`crypto/objects/obj_dat.h`
- **EVP PBE 注册表**：`crypto/evp/evp_pbe.c` 新增 RC2 相关 PBE 条目
- **Legacy Provider**：`providers/legacyprov.c`、`providers/implementations/include/prov/names.h`、`providers/implementations/include/prov/implementations.h` 新增 RC2 注册
- **依赖**：仅影响 Legacy Provider，启用时需显式加载（`-provider legacy`）；RC2 已被认为不安全，不影响默认安全配置
