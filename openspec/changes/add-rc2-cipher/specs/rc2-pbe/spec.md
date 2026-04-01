## ADDED Requirements

### Requirement: PKCS#12 PBE RC2 算法注册
系统 SHALL 在 EVP PBE 注册表中注册以下基于 RC2 的 PKCS#12 PBE 算法：
- `pbeWithSHA1And128BitRC2-CBC`（OID 1.2.840.113549.1.12.1.5），使用 RC2-CBC cipher 和 SHA1 摘要
- `pbeWithSHA1And40BitRC2-CBC`（OID 1.2.840.113549.1.12.1.6），使用 RC2-40-CBC cipher 和 SHA1 摘要

两者 SHALL 使用 `PKCS12_PBE_keyivgen` / `PKCS12_PBE_keyivgen_ex` 作为密钥派生函数。

#### Scenario: pbeWithSHA1And40BitRC2-CBC 注册
- **WHEN** 系统初始化 EVP PBE 注册表
- **THEN** `NID_pbe_WithSHA1And40BitRC2_CBC` SHALL 映射到 `{cipher: NID_rc2_40_cbc, md: NID_sha1, keygen: PKCS12_PBE_keyivgen}`

#### Scenario: pbeWithSHA1And128BitRC2-CBC 注册
- **WHEN** 系统初始化 EVP PBE 注册表
- **THEN** `NID_pbe_WithSHA1And128BitRC2_CBC` SHALL 映射到 `{cipher: NID_rc2_cbc, md: NID_sha1, keygen: PKCS12_PBE_keyivgen}`

### Requirement: 解析使用 RC2-PBE 加密的 PKCS#12 文件
当 Legacy Provider 已加载时，系统 SHALL 能够解析使用 pbeWithSHA1And40BitRC2-CBC 或 pbeWithSHA1And128BitRC2-CBC 加密的 PKCS#12 文件。

#### Scenario: 解析 pbeWithSHA1And40BitRC2-CBC 加密的 PKCS#12
- **WHEN** 加载 Legacy Provider 后，使用 `PKCS12_parse()` 解析一个使用 pbeWithSHA1And40BitRC2-CBC 加密的 .p12 文件
- **THEN** 系统 SHALL 成功解密并返回包含的证书和私钥

#### Scenario: 未加载 Legacy Provider 时解析 RC2-PBE PKCS#12 失败
- **WHEN** 未加载 Legacy Provider 时，使用 `PKCS12_parse()` 解析一个使用 pbeWithSHA1And40BitRC2-CBC 加密的 .p12 文件
- **THEN** 系统 SHALL 返回错误（cipher 不可用）

### Requirement: RC2 PBE OID 注册
系统 SHALL 注册以下 PBE 相关 OID：
- `pbeWithSHA1And128BitRC2-CBC`（OID 1.2.840.113549.1.12.1.5）
- `pbeWithSHA1And40BitRC2-CBC`（OID 1.2.840.113549.1.12.1.6）

#### Scenario: 通过 OID 查找 pbeWithSHA1And40BitRC2-CBC
- **WHEN** 使用 OID 1.2.840.113549.1.12.1.6 查找 NID
- **THEN** 系统 SHALL 返回 `NID_pbe_WithSHA1And40BitRC2_CBC`

#### Scenario: 通过 OID 查找 pbeWithSHA1And128BitRC2-CBC
- **WHEN** 使用 OID 1.2.840.113549.1.12.1.5 查找 NID
- **THEN** 系统 SHALL 返回 `NID_pbe_WithSHA1And128BitRC2_CBC`
