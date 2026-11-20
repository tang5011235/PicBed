# PicBed

个人图床：图片存在本仓库，通过 jsDelivr 的 GitHub CDN 访问。公开仓库，任何拿到链接的人都能看图。

## 链接格式

```
https://cdn.jsdelivr.net/gh/tang5011235/PicBed@main/img/2026/11/20261120-153012-9f3c2a1b.png
```

## 目录规范

```
img/YYYY/MM/<YYYYMMDD-HHMMSS>-<8位随机>.<ext>
```

按年月分目录，避免单目录堆几万个文件拖慢 git 索引。

## 上传

```bash
pic shot.png                  # 打印直链
pic --md shot.png             # 打印 ![](url)，直接粘进笔记
pic --name logo svg-out.png   # 指定文件名（自动补扩展名）
pic --dir img/projects a.png  # 指定子目录
pic --sha big.png             # 额外输出钉住 commit 的链接
pic --dry-run *.png           # 只看计划
```

脚本本体在 `tools/pic`，本机已软链到 `~/.local/bin/pic`。换机器时克隆本仓库，再执行：

```bash
ln -sf ~/development/picbed/tools/pic ~/.local/bin/pic
```

## 限制与已知行为

- 仓库必须**公开**：jsDelivr 只能读公开仓库，私有图只能自己看，走不了这条 CDN。
- 单文件上限约 **20 MB**（jsDelivr 侧），GitHub 单文件硬上限 100 MB，脚本在 20 MB 处拦截。
- jsDelivr 缓存分支引用：**新路径**一般即时可取；**覆盖同名文件**可能要等缓存过期（最长约 12 小时）。要立刻刷新，用 `pic --sha` 打出的 commit 钉住链接，或等缓存过期。
- 免费 CDN + 公开仓库不适合私密图片，也不宜当高流量分发渠道（GitHub ToS）。
- 国内访问速度取决于 jsDelivr 的线路，实测记录见 `docs/verification.md`。
