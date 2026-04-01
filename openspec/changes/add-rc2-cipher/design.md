## Context

Tongsuo 是一个基于 OpenSSL 的密码学库分支。在 Tongsuo 的当前版本中，RC2 算法的所有相关代码（包括 `crypto/rc2/` 底层实现、`include/openssl/rc2.h` 头文件、Provider 层封装、OID 定义等）已被完全移除。

然而，在实际应用中，许多旧版系统和工具（如 Windows Certificate Export Wizard、macOS Keychain、Java KeyStore）在导出 PKCS#12 文件时使用 pbeWithSHA1And40BitRC2-CBC（OID 1.2.840.113549.1.12.1.6）对证书包进行加密。Tongsuo 无法解析这类文件，导致互操作性问题。

Tongsuo 采用 OpenSSL 3.x 的 Provider 架构，密码算法通过 Provider 注册和调度。Legacy Provider 专门用于承载已被认为不安全但仍需兼容的算法（如 DES、RC4、RC5）。RC2 的安全级别与这些算法相当，适合放入 Legacy Provider。

## Goals / Non-Goals

**Goals:**
- 提供完整的 RC2 对称加密底层实现（从 OpenSSL 移植），支持 ECB、CBC、OFB、CFB 四种工作模式
- 支持 RC2 的可变有效密钥长度（40/64/128 位），配合固定的 128 位输入密钥
- 通过 Legacy Provider 注册 RC2 cipher，使其可通过 EVP 接口使用
- 注册 RC2 相关 OID，使 ASN.1 解析可以正确识别 RC2 算法
- 注册基于 RC2 的 PKCS#12 PBE 算法，使 Tongsuo 能够解析使用 RC2-PBE 加密的 PKCS#12 文件
- RC2 默认可用（不在 Configure 中默认禁用），但仅通过 Legacy Provider 提供

**Non-Goals:**
- 不提供 RC2 的汇编优化实现（仅 C 实现）
- 不将 RC2 放入 Default Provider 或 FIPS Provider
- 不提供 RC2 的 AEAD 模式或其他非标准模式
- 不推荐在新系统中使用 RC2（仅用于兼容性目的）
- 不修改现有的安全策略或默认加密套件

## Decisions

### 1. 将 RC2 放入 Legacy Provider 而非 Default Provider

**选择**: 将 RC2 注册到 Legacy Provider  
**理由**: RC2 的有效密钥长度（特别是 40 位变体）完全不符合现代安全标准。与 DES、RC4 等算法一致，放入 Legacy Provider 确保用户必须显式加载才能使用，不会意外降低安全性。  
**备选**: 放入 Default Provider — 被否决，因为这会让不安全的算法在默认配置下可用。

### 2. 从 OpenSSL 移植底层实现

**选择**: 直接从 OpenSSL 移植 `crypto/rc2/` 目录下的源代码  
**理由**: OpenSSL 的 RC2 实现经过长期验证，与 Tongsuo 的代码风格和构建系统兼容。RC2 算法已固定（RFC 2268），无需重新实现。  
**备选**: 从头实现 — 被否决，额外工作量无额外收益且增加引入 bug 的风险。

### 3. RC2 可变有效密钥长度的处理方式

**选择**: 使用 `PROV_CIPHER_FLAG_VARIABLE_LENGTH` 标志，通过 `rc2-cbc`、`rc2-40-cbc`、`rc2-64-cbc` 等不同算法名区分有效密钥位数  
**理由**: 这与 OpenSSL 的 RC2 Provider 设计一致，PBE 算法可通过算法名直接定位到正确的有效密钥长度配置。  
**备选**: 仅通过参数设置有效密钥长度 — 被否决，因为 PBE/PKCS#12 解析依赖 OID-to-cipher 映射，需要独立的算法名。

### 4. 不在 Configure 中默认禁用 RC2

**选择**: 将 `rc2` 加入 `@disablables` 列表和自动检测列表，但不加入默认禁用列表  
**理由**: RC2 已通过 Legacy Provider 的隔离机制保护。默认禁用会导致编译时完全排除 RC2 代码，即使显式加载 Legacy Provider 也无法使用。RC4 和 DES 采用相同策略（默认启用但仅在 Legacy Provider 中）。

## Constraints

- **Out-of-source 构建**：编译测试时 SHALL 使用独立的 `build` 目录进行 out-of-source 构建（如 `mkdir build && cd build && ../Configure && make`），不得在源码根目录中产生构建产物，避免污染源码目录。

## Risks / Trade-offs

- **[安全感知]** 引入 RC2 可能被视为降低安全标准 → 通过 Legacy Provider 隔离缓解；文档明确标注 RC2 为不安全算法
- **[代码维护]** 新增约 1500 行 C 代码需要长期维护 → RC2 算法已固定，不会有功能变更；代码直接来自 OpenSSL 上游，可跟踪上游 bug fix
- **[二进制大小]** 增加 liblegacy 的体积 → 影响极小（RC2 实现非常紧凑）；不影响 libcrypto 主库
- **[OID 冲突]** 新增 OID 的 NID 编号需避免与现有条目冲突 → 使用 `obj_mac.num` 中下一个可用编号，通过 `make update` 重新生成
