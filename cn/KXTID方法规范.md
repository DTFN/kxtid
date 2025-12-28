# KXTID方法规范

## 1. 引言

KXTID 是一种基于国家区块链构建的去中心化标识符（DID）方法，旨在为碳数据提供安全、可信的数字身份。

## 2. DID方法名称

- 注册的方法为 kxtid。
- 有效的标识符以`did:kxtid`为前缀，在前缀之后的DID剩余部分由特定的算法生成。

## 3. DID语法

一个`did:kxtid`的DID由方法前缀和一个可信碳的唯一标识符构成。一个有效的标识符KXTID的ABNF定义如下：


| DID语法定义                                  |
| ---------------------------------------------- |
| `did-kxtid = "did:kxtid:" kxtid-specific-id` |
| `kxtid-specific-id = "0x" 40(pid)`           |
| `pid = ALPHA / DIGIT`                        |

生成KXTID标识符的步骤如下：

1. 使用SM2加密算法
2. 生成公私钥
3. 使用Base64编码方式对公钥进行编码
4. 使用SM3哈希算法对编码后的公钥进行哈希计算
5. 截取哈希值的后40位字符作为位唯一标识
6. 拼接`did:kxtid`、`0x`和唯一标识，形成完整的KXTID

## 4. KXTID文档规范

### 4.1 KXTID规范说明

KXTID文档遵循DID Document规范，并在其基础上做了一定的扩展。KXTID文档字段说明如下：

* context
  数据类型：list<string>
  是否必填：是
  说明：一组解释 JSON-LD 文档的规则，遵循 DID 规范，用于实现不同 DID Document 的互操作，必须包含 https://www.w3.org/ns/did/v1。
* version
  数据类型：string
  是否必填：是
  说明：文档版本号。
* id
  数据类型：string
  是否必填：是
  说明：KXTID 文档的唯一标识符。
* controller
  数据类型：list<string>
  是否必填：是
  说明：一组KXTID，拥有对KXTID文档进行修改、删除等权限。
* authentication
  数据类型：list<string>
  是否必填：是
  说明：一组公钥的KXTID，拥有此公钥对应私钥的一方可以对KXTID文档进行控制和管理。
* verificationMethod
  数据类型：list<VerificationMethod>
  是否必填：是
  说明：一组验证方法，VerificationMethod的字段包括：

  * id
    数据类型：string
    是否必填：是
    说明：验证方法的唯一标识。
  * type
    数据类型：string
    是否必填：是
    说明：验证方法类型。
  * controller
    数据类型：string
    是否必填：是
    说明：验证方法类型。
  * publicKeyMultibase
    数据类型：string
    是否必填：否
    说明：多基编码的公钥。
* service
  数据类型：list<Service>
  是否必填：否
  说明：一组服务地址。Service的字段包括：

  * id
    数据类型：string
    是否必填：是
    说明：服务地址ID。
  * type
    数据类型：string
    是否必填：是
    说明：服务类型标识。
  * serviceEndpoint
    数据类型：string
    是否必填：是
    说明：服务的访问地址。
* proof
  数据类型：list<Proof>
  是否必填：否
  说明：文档所有者对文档内容的签名。Proof的字段包括：

  * created
    数据类型：string
    是否必填：是
    说明：创建时间。
  * proofPurpose
    数据类型：string
    是否必填：是
    说明：签名目的。
  * proofValue
    数据类型：string
    是否必填：是
    说明：签名结果。
  * type
    数据类型：string
    是否必填：是
    说明：签名方法类型。
  * verificationMethod
    数据类型：string
    是否必填：是
    说明：声明用于验证签名的公钥。
* extension

  数据类型：List<Extension>

  是否必须：否

  说明：扩展字段，额外的元数据信息。

  * type

    数据类型：string

    是否必填：是

    说明：扩展类型，自定义。
  * values

    数据类型：list<string>

    是否必填：是

    说明：扩展类型的说明项。
* created
  数据类型：string
  是否必填：是
  说明：KXTID文档的创建时间。
* updated
  数据类型：string
  是否必填：是
  说明：KXTID文档的修改时间。

### 4.2 KXTID文档数据结构定义

KXTID文档数据机构定义如下：


| 字段                 | 数据类型 | 是否必填 | 说明                                                                                                                   |
| ---------------------- | ---------- | ---------- | ------------------------------------------------------------------------------------------------------------------------ |
| `context`            | list     | 是       | 一组解释 JSON-LD 文档的规则，遵循 DID 规范，用于实现不同 DID Document 的互操作，必须包含`https://www.w3.org/ns/did/v1` |
| `version`            | string   | 是       | 文档版本号                                                                                                             |
| `id`                 | string   | 是       | KXTID 文档的唯一标识符                                                                                                 |
| `controller`         | list     | 是       | 一组KXTID，拥有对KXTID文档进行修改、删除等权限                                                                         |
| `authentication`     | list     | 是       | 一组公钥的KXTID，拥有此公钥对应私钥的一方可以对KXTID文档进行控制和管理                                                 |
| `verificationMethod` | list     | 是       | 一组验证方法，字段详情见下方                                                                                           |
| `service`            | list     | 否       | 一组服务地址，字段详情见下方                                                                                           |
| `proof`              | list     | 否       | 文档所有者对文档内容的签名，字段详情见下方                                                                             |
| `extension`          | list     | 否       | 扩展字段，包括额外的元数据                                                                                             |
| `created`            | string   | 是       | KXTID文档的创建时间                                                                                                    |
| `updated`            | string   | 是       | KXTID文档的修改时间                                                                                                    |

#### VerificationMethod 字段说明


| 字段                 | 数据类型 | 是否必填 | 说明               |
| ---------------------- | ---------- | ---------- | -------------------- |
| `id`                 | string   | 是       | 验证方法的唯一标识 |
| `type`               | string   | 是       | 验证方法类型       |
| `controller`         | string   | 是       | 验证方法的控制器   |
| `publicKeyMultibase` | string   | 否       | 多基编码的公钥     |

#### Service 字段说明


| 字段              | 数据类型 | 是否必填 | 说明           |
| ------------------- | ---------- | ---------- | ---------------- |
| `id`              | string   | 是       | 服务地址ID     |
| `type`            | string   | 是       | 服务类型标识   |
| `serviceEndpoint` | string   | 是       | 服务的访问地址 |

#### Proof 字段说明


| 字段                 | 数据类型 | 是否必填 | 说明                   |
| ---------------------- | ---------- | ---------- | ------------------------ |
| `created`            | string   | 是       | 创建时间               |
| `proofPurpose`       | string   | 是       | 签名目的               |
| `proofValue`         | string   | 是       | 签名结果               |
| `type`               | string   | 是       | 签名方法类型           |
| `verificationMethod` | string   | 是       | 声明用于验证签名的公钥 |

### 4.3 KXTID文档示例

kxtid完整的文档示例如下所示：

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1"
  ],
  "version": "1.0",
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
      "type": "Ed25519VerificationKey2018",
      "controller": "did:kxtid:0x7e2f9d4c8a1b3e5f7d9c2b4a6e8f0d1e2f3c4d5b",
      "publicKeyMultibase": "z6MkqRYqQiSgvZQdnBytw86Qbs2ZWUkGv22od935YF4s8M7V"
    }
  ],
  "service": [
    {
      "id": "did:kxtid:0x7e2f9d4c8a1b3e5f7d9c2b4a6e8f0d1e2f3c4d5b#service-1",
      "type": "IssuerService",
      "serviceEndpoint": "http://192.168.3.149/api/v1/query"
    }
  ],
  "proof": [
    {
      "created": "2025-12-19T10:30:00+08:00",
      "proofPurpose": "assertionMethod",
      "proofValue": "MEUCIQCegqh8P+fyY+7UmUkd2FLw9Bbo8ILdGAMFwzlqrcjXNAIgfLPAXDvQgPWeQI4lLdlM2nvldEYw6DQpp/NgWn8w4pQ=",
      "type": "Ed25519VerificationKey2018",
      "verificationMethod": "did:kxtid:0x7e2f9d4c8a1b3e5f7d9c2b4a6e8f0d1e2f3c4d5b#key-1"
    }
  ],
  "created": "2025-12-19T10:00:00+08:00",
  "updated": "2025-12-19T10:30:00+08:00"
}
```

## 5. 方法操作

DID方法包括创建、读取、更新和停用。

### 5.1 创建（Created）

创建接口主要完成KXTID文档的创建，支持HTTP POST 方法。创建KXTID 文档时，proof字段的签名者需要是 authentication字段里的 公钥才有权限创建成功，如果KXTID文档存在，则不允许重复创建。

### 5.2 读取（Read）

读取接口主要完成KXTID文档的查询，支持HTTP GET方法。返回值为KXTID文档的 JSON 字符串。

### 5.3 更新（Update）

更新接口主要完成KXTID 文档的更新，支持HTTP POST 方法。更新操作不允许更新 authentication字段，更新 KXTID 文档时 proof 字段的签名者需要是 authentication字段里的 公钥才有权限更新成功。

### 5.4 停用（Deactivate）

停用接口主要完成对 KXTID文档停用，支持 HTTP POST方法。停用后的 KXTID文档更新为空而不是删除。

## 6. 安全和隐私

### 6.1 安全

- **公钥完整性**：KXTID的公钥存储在区块链上，确保其不可篡改。
- **私钥保护**：私钥必须安全存储，避免泄露给未经授权的第三方。
- **防止中间人攻击**：通过加密传输（如 TLS）保护KXTID文档的传输过程。

### 6.2 隐私

- **数据最小化**：仅在区块链上存储必要的信息，如公钥和服务端点，避免存储敏感信息。
- **访问控制**：通过加密方法控制对 KXTID 及其关联服务的访问权限。
