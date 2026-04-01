## ADDED Requirements

### Requirement: RC2 底层加密实现
系统 SHALL 提供 RC2 分组密码的底层实现，包括密钥调度（key schedule）和分组加/解密操作。RC2 的分组大小 SHALL 为 64 位（8 字节），输入密钥长度 SHALL 为 128 位（16 字节）。

#### Scenario: RC2 密钥调度
- **WHEN** 使用 16 字节密钥和指定的有效密钥位数（如 128、64、40）调用 RC2 密钥调度函数
- **THEN** 系统 SHALL 生成正确的 RC2 密钥调度表

#### Scenario: RC2 单块加密
- **WHEN** 使用有效的 RC2 密钥调度表对 8 字节明文块调用 RC2 加密函数
- **THEN** 系统 SHALL 输出正确的 8 字节密文块

#### Scenario: RC2 单块解密
- **WHEN** 使用有效的 RC2 密钥调度表对 8 字节密文块调用 RC2 解密函数
- **THEN** 系统 SHALL 输出正确的 8 字节明文块

### Requirement: RC2 工作模式支持
系统 SHALL 支持 RC2 的以下工作模式：CBC（Cipher Block Chaining）、ECB（Electronic Codebook）、OFB（Output Feedback，64 位）、CFB（Cipher Feedback，64 位）。

#### Scenario: RC2 CBC 模式加解密
- **WHEN** 通过 EVP 接口使用 RC2-CBC 模式（128 位有效密钥）加密数据
- **THEN** 系统 SHALL 生成符合 RC2-CBC 标准的密文，且使用相同密钥和 IV 解密后得到原始明文

#### Scenario: RC2 ECB 模式加解密
- **WHEN** 通过 EVP 接口使用 RC2-ECB 模式加密数据
- **THEN** 系统 SHALL 生成符合 RC2-ECB 标准的密文

#### Scenario: RC2 OFB 模式加解密
- **WHEN** 通过 EVP 接口使用 RC2-OFB 模式加密数据
- **THEN** 系统 SHALL 生成符合 RC2-OFB 标准的密文

#### Scenario: RC2 CFB 模式加解密
- **WHEN** 通过 EVP 接口使用 RC2-CFB 模式加密数据
- **THEN** 系统 SHALL 生成符合 RC2-CFB 标准的密文

### Requirement: RC2 可变有效密钥长度
系统 SHALL 支持 RC2 的 40 位、64 位和 128 位有效密钥长度。系统 SHALL 提供独立的算法名以区分不同的有效密钥长度：`RC2-CBC`（128 位）、`RC2-40-CBC`（40 位）、`RC2-64-CBC`（64 位）。

#### Scenario: RC2-40-CBC 加解密
- **WHEN** 使用 `RC2-40-CBC` 算法名获取 cipher 并执行加解密
- **THEN** 系统 SHALL 使用 40 位有效密钥长度进行 RC2 密钥调度，且加解密结果正确

#### Scenario: RC2-64-CBC 加解密
- **WHEN** 使用 `RC2-64-CBC` 算法名获取 cipher 并执行加解密
- **THEN** 系统 SHALL 使用 64 位有效密钥长度进行 RC2 密钥调度，且加解密结果正确

### Requirement: RC2 通过 Legacy Provider 注册
RC2 cipher 算法 SHALL 仅通过 Legacy Provider 注册和提供。默认配置下（未加载 Legacy Provider），RC2 SHALL 不可用。

#### Scenario: 默认配置下 RC2 不可用
- **WHEN** 未加载 Legacy Provider 时通过 EVP 接口请求 RC2-CBC
- **THEN** 系统 SHALL 返回空指针（算法未找到）

#### Scenario: 加载 Legacy Provider 后 RC2 可用
- **WHEN** 显式加载 Legacy Provider 后通过 EVP 接口请求 RC2-CBC
- **THEN** 系统 SHALL 返回有效的 EVP_CIPHER 对象

### Requirement: RC2 OID 注册
系统 SHALL 注册以下 RC2 相关 OID：
- `RC2-CBC`（OID 1.2.840.113549.3.2）
- `RC2-ECB`（OID 1.2.840.113549.3.3，如适用）
- `RC2-40-CBC`（无独立 OID，通过参数区分）
- `RC2-64-CBC`（无独立 OID，通过参数区分）

#### Scenario: 通过 OID 查找 RC2-CBC
- **WHEN** 使用 OID 1.2.840.113549.3.2 查找 NID
- **THEN** 系统 SHALL 返回 RC2-CBC 对应的 NID

### Requirement: 构建系统集成
系统 SHALL 在构建系统中正确集成 RC2：
- `Configure` 中 `rc2` SHALL 作为可禁用选项存在
- `crypto/build.info` 中 SHALL 包含 `rc2` 子目录
- `providers/implementations/ciphers/build.info` 中 SHALL 将 RC2 cipher 文件链接到 `liblegacy.a`
- 当 `no-rc2` 被指定时，所有 RC2 代码 SHALL 被排除

#### Scenario: 默认构建包含 RC2
- **WHEN** 使用默认配置（不指定 `no-rc2`）执行构建
- **THEN** RC2 代码 SHALL 被编译并包含在 liblegacy 中

#### Scenario: 禁用 RC2 构建
- **WHEN** 使用 `no-rc2` 配置选项执行构建
- **THEN** 所有 RC2 代码 SHALL 被排除，`OPENSSL_NO_RC2` 宏 SHALL 被定义
