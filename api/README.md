# api/ — 机读区

本目录由 LiveDeck 客户端读取，用于检查版本更新与公告。

**请勿手工编辑本目录下的任何文件。** 全部内容由源码仓库的
`pnpm run release:manifest` 生成并签名，手改会导致签名验证失败，客户端将拒绝
使用整份清单。

## 文件

| 文件 | 说明 |
| --- | --- |
| `manifest.json` | 唯一入口。客户端只认这个文件的地址，其余全部通过它发现。 |
| `manifest.json.sig` | 上述文件的 Ed25519 签名。 |
| `public-keys.json` | 验签公钥。与客户端内置的是同一份，此处公开供第三方独立验证。 |
| `releases/<版本>.json` | 单个版本的更新内容与下载资源（含 SHA256）。 |
| `notices/<编号>.json` | 单条公告的正文。 |

## 完整性

`manifest.json` 经 Ed25519 签名，签名私钥离线保管，不在任何仓库中。

清单内记录了 `releases/` 与 `notices/` 下每个文件的 SHA256，这些文件内又记录了
每个二进制资源的 SHA256 —— 因此一个签名即可传递保护整条链。客户端在验签失败时
会丢弃该来源的结果并尝试下一个镜像，而不是接受未经验证的数据。

任何人都可以独立验证：

```bash
# 需要 Node.js
node -e "
const c=require('node:crypto'),f=require('node:fs');
const m=f.readFileSync('manifest.json');
const s=JSON.parse(f.readFileSync('manifest.json.sig','utf8'));
const k=JSON.parse(f.readFileSync('public-keys.json','utf8'));
const key=k.keys.find(x=>x.id===s.keyId);
const pub=c.createPublicKey({key:Buffer.from(key.publicKey,'base64'),format:'der',type:'spki'});
console.log(c.verify(null,m,pub,Buffer.from(s.signature,'base64'))?'签名有效':'签名无效');
"
```

## 缓存

客户端请求时会带一个按小时取整的 `ts` 查询参数，用于在保留 CDN 缓存收益的同时把
陈旧窗口压到一小时以内。因此新发布的清单最长约一小时后才会被全部客户端看到。
