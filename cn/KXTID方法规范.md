# KXTID方法规范

## 1. 引言

`did:kxtid`方法是一种基于区块链技术构建的去中心化标识符（DID）方法，旨在为碳数据提供安全、可信的身份标识服务。`did:kxtid`使用区块链做为可验证数据注册表（VDR）进行DID文档的存储，能有效保障数据的安全性。

## 2. DID方法名称

- 注册的方法为 kxtid。
  
- 有效的标识符以`did:kxtid`为前缀，在前缀之后的DID剩余部分由特定的算法生成。
  

## 3. DID语法

一个`did:kxtid`的DID由方法前缀和一个可信碳的唯一标识符构成。一个有效的标识符KXTID的ABNF定义如下：

| DID语法定义 |
| --- |
| `did-kxtid = "did:kxtid:" kxtid-specific-id` |
| `kxtid-specific-id = "0x" 40(hexdig)` |
| `hexdig = ALPHA / DIGIT` |

`did:kxtid` 的DID是基于椭圆密码学（Elliptic Curve Cryptography，ECC）进行生成的，其具体生成步骤如下：

1. 选择算法：选择SM2 算法
  
2. 生成私钥：生成256位（32字节）随机私钥
  
3. 推导公钥：基于私钥推导非压缩格式的公钥（65字节，前缀为04）
  
4. 哈希处理：非压缩格式的公钥移除前缀`04`，并使用 SM3杂凑算法对其进行哈希计算，获得32字节的哈希值
  
5. 标识符拼接：取32字节哈希值的后20位字节，并将其转为40位十六进制的字符串，然后拼接`0x`前缀，从而形成完整的KXTID标识，示例：`0x7e2f9d4c8a1b3e5f7d9c2b4a6e8f0d1e2f3c4d5b`
  

**重要说明：**

1.SM2是由中国国家密码管理局制定的独立国产椭圆曲线国密标准，其对应标准为 GM/T 0003-2012（等同ECDSA secp256k1算法）

2.SM3是由中国国家密码管理局制定的独立国产杂凑算法标准，其对应标准为GB/T 32905-2016（等同Keccak-256哈希算法）

## 4. 方法操作

DID方法包括创建、读取、更新和停用。

### 4.1 创建（Created）

该方法用于创建DID以及DID对应文档，并将DID存储在可验证数据注册表（VDR），即存储到区块链上。

1.按照`did:kxtid`的DID的生成流程，生成DID；

2.使用生成的DID创建DID文档，DID文档的字段信息简要说明如下：

- 包含 `@context`、`id`、`verificationMethod` 、`authentication` 字段；
- 可选包括`service`、`extension`、`version`、`created`和`updated`字段。

3.调用区块链的智能合约，将DID文档注册到区块链上。

**重要说明：**

1.该区块链为中国境内搭建的一条可信碳企业联盟链，用于企业碳数据的共享。

2.`extension`、`version`、`created`和`updated` 为新增字段，其字段含义如下：

- `extension`
  
  数据类型：Extension
  
  是否必须：否
  
  说明：扩展字段，额外的业务元数据信息，其包括`children`字段属性。
  
- `version`
  
  数据类型：string
  是否必填：否
  说明：KXTID文档的版本，如：1.0,2.0。
  
- created
  数据类型：string
  是否必填：否
  说明：KXTID文档的创建时间。
  
- updated
  数据类型：string
  是否必填：否
  说明：KXTID文档的修改时间。
  

### 4.2 读取（Read）

该方法支持通过DID解析并读取对应DID文档。

`did:kxtid`的读取是通过`KXTID解析器`服务完成的。解析器通过调用智能合约从区块链上查询DID文档，并返回给读取方。解析详情请参考第5章节。

### 4.3 更新（Update）

该方法用于更新已存在的DID文档。

1.准备好待更新的DID文档（允许更新字段为：`verificationMethod`、`authentication`、`service`、`extension`、`version`、`updated`）

2.根据DID调用区块链的智能合约，查找其对应的链上的DID文档

3.验证持有者与链上的DID文档中`controller` 和 `verificationMethod`中的`publicKeyHex` 信息是否一致

**重要说明：**

1.可通过更新 DID 文档添加新的 `verificationMethod`完成密钥轮换。新增密钥时，可保留现有密钥；若密钥泄露，可以移除旧密钥，并通过更新DID文档中`verificationMethod`字段使用新密钥。

### 4.4 停用（Deactivate）

该方法用于停用已存在的DID文档。

1.根据DID调用区块链的智能合约，查找其对应的DID文档；

2.验证持有者与DID文档中`controller` 和 `verificationMethod`中的`publicKeyHex` 信息是否一致

2.若一致，则将其DID文档为空。

**重要说明：**

1.DID文档停用后，无法被读取和更新。

## 5.DID解析

基于HTTP方式提供DID解析服务。解析通过向DID解析器发送GET请求进行解析执行。

**接口地址**

`GET /1.0/identifiers/{did}`

**处理流程**

- 调用区块链的智能合约，根据DID查询链上的DID文档
  
- 若找到DID文档，返回`200 OK`，并返回DID文档（格是为：`application/ld+json`）
  
- 若未找到DID文档，返回 `404 notFound` 错误
  

**响应说明**

DID文档的示例如下所示：

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1"
  ],
  "id": "did:kxtid:0x7e2f9d4c8a1b3e5f7d9c2b4a6e8f0d1e2f3c4d5b",
  "controller": [
    "did:kxtid:0x7e2f9d4c8a1b3e5f7d9c2b4a6e8f0d1e2f3c4d5b"
  ],
  "authentication": [
    "did:kxtid:0x7e2f9d4c8a1b3e5f7d9c2b4a6e8f0d1e2f3c4d5b#key-1"
  ],
  "verificationMethod": [
    {
      "id": "did:kxtid:0x7e2f9d4c8a1b3e5f7d9c2b4a6e8f0d1e2f3c4d5b#key-1",
      "type": "EcdsaSecp256k1VerificationKey2019",
      "controller": "did:kxtid:0x7e2f9d4c8a1b3e5f7d9c2b4a6e8f0d1e2f3c4d5b",
      "publicKeyHex": "04611176548da4a758187950dc2708a58ff7b4978c38cd9ae0961b889b74fa2f7be8592d69592bb2ffbc5315f8a563f479671ecf422d2ebd38d38c84c0d451b65c"
    }
  ],
  "extension": {
     "children":[
        "did:kxtid:0x8e2f9d4c8a1b3e5f7d9c2b4a6e8f0d1e2f3c4d5b",
        "did:kxtid:0x9e2f9d4c8a1b3e5f7d9c2b4a6e8f0d1e2f3c4d6b",
     ]
  },
  "version": "1.0",
  "created": "2025-12-19T10:00:00+08:00",
  "updated": "2025-12-19T10:30:00+08:00"
}
```

## 6. 安全和隐私

### 6.1 安全

- **私钥保护**：私钥必须安全存储，避免泄露给未经授权的第三方。
- **防止中间人攻击**：通过加密传输（如 TLS）保护KXTID文档的传输过程。

### 6.2 隐私

- **数据最小化**：仅在区块链上存储必要的信息，避免存储敏感信息。
- **访问控制**：通过加密方法控制对 KXTID 及其关联服务的访问权限。
