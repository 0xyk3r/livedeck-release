# api/ — 机读区

本目录提供 LiveDeck 的版本清单、更新说明、公告与下载信息。

**JSON 文件与签名由源码仓库生成，请勿手工修改。** 两个仓库并排检出时，在源码仓库运行 `pnpm run release:manifest --out ../livedeck-release` 生成并签名，用 `pnpm run release:verify --dir ../livedeck-release` 离线校验。只改本 README 或用户文档不需要重新签名。

## 文件

| 文件 | 说明 |
| --- | --- |
| `manifest.json` | 清单入口，包含稳定版本、最低支持版本、历史版本、公告及可用端点 |
| `manifest.json.sig` | 清单原始字节的 Ed25519 签名 |
| `public-keys.json` | 公开公钥表，供核对与独立验签；客户端使用构建期内置的可信公钥 |
| `releases/<版本>.json` | 单个版本的更新说明与资源名称、大小、SHA256、下载地址 |
| `notices/<编号>.json` | 单条公告的结构化正文 |

版本说明与公告的 `detail` 相对于 `api/`。先验签 `manifest.json` 的原始字节，再按对应的 `detailSha256` 校验正文；读取新字段时应兼容旧格式。历史版本的下载地址记录在各自的说明中，不一定与稳定版的镜像列表相同。

## 完整性与可信公钥

清单内记录版本说明与公告正文的 SHA256，版本说明又记录二进制资源的 SHA256。一个有效签名保护整条摘要链；客户端遇到签名失败会尝试其他端点。

验签必须使用事先可信的公钥。不要把未知镜像返回的清单、签名和公钥表一起当作可信输入，否则镜像可以同时替换三者。请从已核实的发布者来源取得并核对公钥，保存为 `trusted-public-keys.json`。已撤销的密钥不能用于验签。

签名与摘要确认来源及完整性，不代表系统已验证安装包发布者身份；当前 Windows 安装包未使用代码签名证书。

## 独立验签

将 `manifest.json`、`manifest.json.sig` 和已核实的 `trusted-public-keys.json` 放在同一目录。把以下代码保存为 `verify-manifest.mjs`，在该目录运行 `node verify-manifest.mjs`（需要 Node.js）：

```javascript
import fs from 'node:fs'
import crypto from 'node:crypto'

const bytes = fs.readFileSync('manifest.json')
const signature = JSON.parse(fs.readFileSync('manifest.json.sig', 'utf8'))
const { keys } = JSON.parse(fs.readFileSync('trusted-public-keys.json', 'utf8'))
const usable = keys.filter(key => key.status !== 'revoked' && key.algorithm === 'ed25519')
const ordered = [
  ...usable.filter(key => key.id === signature.keyId),
  ...usable.filter(key => key.id !== signature.keyId),
]
const valid =
  signature.algorithm === 'ed25519' &&
  typeof signature.signature === 'string' &&
  signature.signature !== '' &&
  ordered.some(key => {
    try {
      const publicKey = crypto.createPublicKey({
        key: Buffer.from(key.publicKey, 'base64'),
        format: 'der',
        type: 'spki',
      })
      return (
        publicKey.asymmetricKeyType === 'ed25519' &&
        crypto.verify(null, bytes, publicKey, Buffer.from(signature.signature, 'base64'))
      )
    } catch {
      return false
    }
  })

console.log(valid ? '签名有效' : '签名无效')
process.exitCode = valid ? 0 : 1
```

上述命令只验证清单签名。之后仍需按清单中的 SHA256 核对版本说明、公告正文与实际下载文件；安装包核对步骤见[安装说明](../docs/install.md#核对-sha256)。

## 缓存与传播

客户端请求带按小时取整的 `ts` 查询参数，每小时使用新的缓存键。客户端也按自己的检查周期获取更新，第三方 CDN 可能缓存或暂时不可用，因此不能保证发布后一小时内所有客户端都已看到新版。

需要及时确认时可主动检查更新，或访问[官方最新发布页](https://github.com/0xyk3r/livedeck-release/releases/latest)。
