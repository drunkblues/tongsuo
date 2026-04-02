## 1. 构建系统与配置

- [x] 1.1 在 `Configure` 中将 `rc2` 加入 `@disablables` 列表
- [x] 1.2 在 `Configure` 的自动检测列表（`crypto/<alg>/` 目录检测）中添加 `rc2`
- [x] 1.3 确认 `rc2` 不在默认禁用列表中（`%disabled` 初始赋值）
- [x] 1.4 在 `crypto/build.info` 的 `SUBDIRS` 中添加 `rc2`

## 2. RC2 底层实现（crypto/rc2/）

- [x] 2.1 创建 `crypto/rc2/` 目录
- [x] 2.2 从 OpenSSL 移植 `rc2_local.h`（内部数据结构和宏定义）
- [x] 2.3 从 OpenSSL 移植 `rc2_skey.c`（密钥调度函数 `RC2_set_key`）
- [x] 2.4 从 OpenSSL 移植 `rc2_ecb.c`（ECB 模式封装 `RC2_ecb_encrypt`）
- [x] 2.5 从 OpenSSL 移植 `rc2_cbc.c`（CBC 模式 `RC2_cbc_encrypt` 及核心加解密 `RC2_encrypt`/`RC2_decrypt`）
- [x] 2.6 从 OpenSSL 移植 `rc2cfb64.c`（CFB 64 位模式 `RC2_cfb64_encrypt`）
- [x] 2.7 从 OpenSSL 移植 `rc2ofb64.c`（OFB 64 位模式 `RC2_ofb64_encrypt`）
- [x] 2.8 创建 `crypto/rc2/build.info`，将源文件链接到 `../../libcrypto` 和 `../../providers/liblegacy.a`

## 3. 公共头文件

- [x] 3.1 创建 `include/openssl/rc2.h`，定义 `RC2_KEY` 结构体和废弃的低级 API 声明（`RC2_set_key`、`RC2_ecb_encrypt`、`RC2_encrypt`、`RC2_decrypt`、`RC2_cbc_encrypt`、`RC2_cfb64_encrypt`、`RC2_ofb64_encrypt`），使用 `OSSL_DEPRECATEDIN_3_0` 宏标记，并添加 `OPENSSL_NO_RC2` 编译守卫

## 4. OID 注册

- [x] 4.1 在 `crypto/objects/objects.txt` 中添加 RC2 相关 OID 条目（`RC2-CBC`、`RC2-ECB`、`RC2-CFB64`、`RC2-OFB64`、`RC2-40-CBC`、`RC2-64-CBC`）
- [x] 4.2 在 `crypto/objects/objects.txt` 中添加 RC2 PBE 相关 OID 条目（`pbeWithSHA1And128BitRC2-CBC`、`pbeWithSHA1And40BitRC2-CBC`）
- [x] 4.3 在 `crypto/objects/obj_mac.num` 中为上述 OID 分配 NID 编号
- [x] 4.4 运行 `make update` 或对应脚本重新生成 `include/openssl/obj_mac.h` 和 `crypto/objects/obj_dat.h`

## 5. Provider 层实现

- [x] 5.1 创建 `providers/implementations/ciphers/cipher_rc2.h`，定义 `PROV_RC2_CTX` 结构体（包含 `PROV_CIPHER_CTX base` 和 `RC2_KEY` 以及 `unsigned int effective_keybits`），声明各模式的 `PROV_CIPHER_HW` 获取函数
- [x] 5.2 创建 `providers/implementations/ciphers/cipher_rc2.c`，实现 `rc2_newctx`/`rc2_freectx`/`rc2_dupctx`，实现 `rc2_get_ctx_params`/`rc2_set_ctx_params`（处理 effective key bits 参数），使用宏生成 `ossl_rc2128ecb_functions`、`ossl_rc2128cbc_functions`、`ossl_rc2128ofb128_functions`、`ossl_rc2128cfb128_functions`、`ossl_rc240cbc_functions`、`ossl_rc264cbc_functions` 等 dispatch 表
- [x] 5.3 创建 `providers/implementations/ciphers/cipher_rc2_hw.c`，实现 `cipher_hw_rc2_initkey`（调用 `RC2_set_key`）和各模式的 cipher 函数（调用 `RC2_ecb_encrypt`、`RC2_cbc_encrypt`、`RC2_cfb64_encrypt`、`RC2_ofb64_encrypt`），使用宏生成 `PROV_CIPHER_HW` 结构体
- [x] 5.4 在 `providers/implementations/ciphers/build.info` 中添加 `$RC2_GOAL=../../liblegacy.a` 和 RC2 源文件条件编译块

## 6. Provider 注册

- [x] 6.1 在 `providers/implementations/include/prov/names.h` 中添加 `PROV_NAMES_RC2_CBC`、`PROV_NAMES_RC2_ECB`、`PROV_NAMES_RC2_OFB`、`PROV_NAMES_RC2_CFB`、`PROV_NAMES_RC2_40_CBC`、`PROV_NAMES_RC2_64_CBC` 宏定义（包含 OID）
- [x] 6.2 在 `providers/implementations/include/prov/implementations.h` 中添加 `ossl_rc2128ecb_functions[]`、`ossl_rc2128cbc_functions[]`、`ossl_rc2128ofb128_functions[]`、`ossl_rc2128cfb128_functions[]`、`ossl_rc240cbc_functions[]`、`ossl_rc264cbc_functions[]` 的 `extern` 声明，使用 `OPENSSL_NO_RC2` 编译守卫
- [x] 6.3 在 `providers/legacyprov.c` 的 `legacy_ciphers[]` 表中注册所有 RC2 cipher 变体，使用 `OPENSSL_NO_RC2` 编译守卫

## 7. EVP PBE 注册

- [x] 7.1 在 `crypto/evp/evp_pbe.c` 的 `builtin_pbe[]` 表中添加 `pbeWithSHA1And128BitRC2-CBC` 条目（`{EVP_PBE_TYPE_OUTER, NID_pbe_WithSHA1And128BitRC2_CBC, NID_rc2_cbc, NID_sha1, PKCS12_PBE_keyivgen, PKCS12_PBE_keyivgen_ex}`）
- [x] 7.2 在 `crypto/evp/evp_pbe.c` 的 `builtin_pbe[]` 表中添加 `pbeWithSHA1And40BitRC2-CBC` 条目（`{EVP_PBE_TYPE_OUTER, NID_pbe_WithSHA1And40BitRC2_CBC, NID_rc2_40_cbc, NID_sha1, PKCS12_PBE_keyivgen, PKCS12_PBE_keyivgen_ex}`）

## 8. 验证与测试（须在 `build` 目录中进行 out-of-source 构建，不得污染源码目录）

- [x] 8.1 创建 `build` 目录，在其中执行 `../Configure` 确认默认配置下 RC2 未被禁用
- [x] 8.2 在 `build` 目录中执行 `make` 确认编译通过，无错误或警告
- [x] 8.3 清理后在 `build` 目录中使用 `no-rc2` 选项执行 `../Configure` 并 `make`，确认 RC2 被正确排除且编译通过
- [x] 8.4 使用 `openssl list -cipher-algorithms -provider legacy` 验证 RC2 cipher 已注册
- [x] 8.5 使用 RC2-CBC 执行基本加解密测试，验证功能正确性
- [x] 8.6 使用一个 pbeWithSHA1And40BitRC2-CBC 加密的 PKCS#12 测试文件验证解析功能
