# 实测记录

环境：Deepin 23.1 / git 2.47.2 / gh 2.97.0（账号 tang5011235，SSH 协议）/ ImageMagick(convert)
时间：2026-11-20 11:24–11:27（本机时间）

## 1. 建仓

```
gh repo create tang5011235/PicBed --public --add-readme
git clone git@github.com:tang5011235/PicBed.git ~/development/picbed
```

结果：`visibility=PUBLIC branch=main`。

## 2. 新路径在 jsDelivr 上的可见性

首次试传 `img/2026/11/20261120-112431-188bf557.{png,jpg}`（commit `e9a77e0`），推送完成**立刻**请求：

| 链接类型 | T+0 | T+12s |
| --- | --- | --- |
| `@main` .png | 200 image/png 31196B 2.20s | 200 1.76s |
| `@main` .jpg | 200 image/jpeg 21611B 3.74s | 200 1.27s |
| `@e9a77e0950a9` .png | 200 31196B 1.65s | 200 1.12s |
| raw.githubusercontent.com | 200 31196B 1.63s | 200 2.37s |

结论：**新文件路径不需要等缓存，推完即可取**，`@main` 与 commit 钉住链接都可用、返回的 Content-Type 正确。

## 3. 字节一致性

样例 `img/2026/11/picbed-demo.png`（commit `3835b4e`）从 jsDelivr 下载后与本地文件比对：

```
sha256sum /tmp/picbed-test.png /tmp/dl.png | awk '{print $1}' | uniq -c
      2 5f61662a72f0a68d3c33718a44b106580917d81289cfa7cc99a8d898f39d4999
```

同一哈希出现两次，CDN 未做转码/压缩，内容原样返回。

## 4. 速度

同一条链接连续取三次（本机电信线路，波动明显）：

| 次序 | TTFB | 总耗时 |
| --- | --- | --- |
| 1 | 3.80s | 4.81s |
| 2 | 2.95s | 4.08s |
| 3 | 1.83s | 2.24s |

另一批探测在 1.1–1.7s 之间。即国内访问**可用但不稳定，1–5s 区间**。

## 5. 未验证项

- **覆盖同名文件后的缓存过期时间**（README 写的是"最长约 12 小时"，取自 jsDelivr 对分支引用的缓存策略，未在本仓实测）。需要立刻刷新时用 `pic --sha` 的 commit 链接，或把文件名换成新的。
- jsDelivr 单文件 20 MB 上限：只验了脚本在 21 MB 处拦截，没有实测 20 MB 附近边界的服务端行为。

## 复现

```bash
pic --sha /tmp/picbed-test.png          # 上传并同时打印 @main 与 @<sha> 链接
curl -sS -o /tmp/dl.png -w '%{http_code} %{size_download}\n' <url>
sha256sum /tmp/picbed-test.png /tmp/dl.png
```
