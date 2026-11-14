# 归档溯源说明

本仓库是 NapCatQQ 的完整官方历史归档。以下所有结论均经 Git 对象级核实，非推测。

## 一、本仓库保留了什么

除上游最终删库的两个提交之外，保留了官方全部历史：

| 项目 | 内容 |
|---|---|
| 起始提交 | `0427f132 docs:init`，2024-03-26，真正的初始提交 |
| 终止提交 | `72912f293 docs: ending`，2026-11-10，官方公开历史的最后一条 |
| 提交总数 | 5028 条，全在 main 链上，父节点缺失数为 0 |
| tag 总数 | 761 个，v1.0.0 至 v4.18.42，每个 tag 指向的提交均存在于本仓库 |
| 全 ref 可达提交 | 9838 条，即 main 链 5028 条加发布分支与合并链上的 4810 条 |
| 第三方改动 | 0 |

## 二、除去了什么

上游在 2026-11-12 强制推送清空了仓库。这两个提交**不在**本仓库内：

| 提交 | 内容 | 性质 |
|---|---|---|
| `363a595 Keep README only` | 只保留 README | 孤立根提交，父节点数为 0 |
| `105f38e docs: ending` | 清空后的收尾 | 父节点为 `363a595` |

`363a595` 的父节点数为 0，说明官方不是正常提交删除，而是强制推送抹掉了全部历史。因此上游 `NapNeko/NapCatQQ` 当前 `git ls-remote --tags` 返回 0 个 tag，只剩一个 main 指向 `105f38e`。

本仓库以 `72912f29` 为终点，即清库发生前的最后一条官方提交，删库动作本身不纳入。

## 三、为何可以确认这些是官方内容

### 作者构成

main 链 5028 条提交的作者全部为原项目贡献者：

| 作者 | 提交数 |
|---|---|
| 手瓜一十雪 <nanaeonn@outlook.com> | 3462 |
| Mlikiowa <nanaeonn@outlook.com> | 328 |
| linyuchen <lin.yu.chen@hotmail.com> | 237 |
| Wesley F. Young <willamwei@outlook.com> | 188 |
| pk5ls20 <pk5ls20@outlook.com> | 180 |
| bietiaop <1527109126@qq.com> | 83 |
| Alen | 58 + 54 |
| 其余 | 见 `git log --format='%an <%ae>' main \| sort \| uniq -c` |

无任何第三方组织或镜像维护者的提交。

### 终点提交带 GitHub 签名

```
72912f293924f43463b5976b01fc094a8ae11481
 author: MliKiowa <nanaeonn@outlook.com>  2026-11-10 09:06:40 +0800
 commit: GitHub <noreply@github.com>      2026-11-10 09:06:40 +0800
 subject: docs: ending
 body: Signed-off-by: MliKiowa <nanaeonn@outlook.com>
```

由 GitHub 网页端生成并签名，非本地伪造。

### 与多个独立 fork 交叉验证

拿本仓库 main 与四个互不相关的独立 fork 比对最近 300 条提交：

| fork | 共同提交 |
|---|---|
| `Creep-debug/NapCatQQ` | 300 / 300 |
| `xxynet/NapCatQQ` | 300 / 300 |
| `lexy76888-dev/NapCatQQ` | 300 / 300 |
| `MingTianSang/NapCatQQ` | 289 / 300 |

`MingTianSang` 差的 11 条是因为它停在 2026-10-27 的旧版本，其 HEAD `ba0a53d` 经 `git merge-base --is-ancestor` 验证是本仓库 main 的祖先，不存在分叉。

另用 `--bare --filter=blob:none` 完整克隆 `Creep-debug/NapCatQQ` 复核，其 `rev-list --count main` 同为 5028。

### 断链检测

遍历 main 每一条提交，逐个检查其父提交是否可达，结果：**父提交缺失数 0**。

## 四、关于 tag 不在 main 链上的说明

`git rev-list --all` 为 9838 条，比 main 的 5028 条多出 4810 条。原因是 761 个 tag 中有 666 个不指向 main 链上的提交。

这是原项目自身的开发习惯，不是数据异常：

- 多数 tag 打在 PR 分支 squash 合并前的原始提交上，main 上是合并后的另一条提交
- 例：`v4.18.10` 的分叉点为 `7c2514be6`（2026-07-19），此后 tag 侧只有 1 条提交 `8447a46f8 Add mappings for QQ 50969 builds`，而 main 侧前进了 81 条
- 这 663 个非 main 链 tag 提交的作者分布：手瓜一十雪 605、pk5ls20 16、bietiaop 8、mitsukina 5、时瑾 4、linyuchen 4 等，**全部为官方贡献者**

逐个执行 `git cat-file -e` 验证，761 个 tag 指向的提交**缺失 0 个**。所以这部分提交是原项目本来就存在的，不是任何第三方注入。

## 五、实现方式

本仓库未通过 GitHub Fork 功能创建，是一个独立仓库，页面上没有 `forked from` 标记，上游仓库即使再次删除也不影响本仓库。

搬运在 GitHub Actions 的服务端完成，因为本地设备磁盘空间不足以承载完整推送：

1. 在 runner 上 `git clone --bare --filter=blob:none` 归档源仓库
2. 用 PAT（`GITHUB_TOKEN` 缺少 `workflows` 权限，无法推送 workflow 文件）执行 `git push --force` 回本仓库
3. 校验 tree hash 与提交数一致后切换默认分支

> 说明：`--filter=blob:none` 只影响本地工作副本按需下载文件内容，不影响仓库本身的完整性与全部历史对象。

## 六、自行复核

以下命令可在任意克隆中复现上述结论：

```bash
git rev-list --count main                  # 5028
git log --format='%an <%ae>' main | sort | uniq -c   # 作者分布
git rev-list --count --all                 # 9838
git tag | wc -l                            # 761
git log -1 --format='%H%n%an <%ae>%n%s' 72912f29    # 终点提交

# 断链检测：应无输出
git rev-list main | while read c; do
  for p in $(git log -1 --format=%p "$c"); do
    git cat-file -e "$p^{commit}" 2>/dev/null || echo "缺失: $c -> $p"
  done
done
```

## 七、许可与声明

代码版权归 NapCat 原作者及贡献者所有，本仓库仅为历史归档，不对代码内容作任何修改主张。使用须遵守仓库内 [LICENSE](./LICENSE) 与上游声明。

上游原话（保留于 README 末尾）：如果感兴趣可以自行 Fork，不再提供除版本更新以外内容，直到失效，如果追求稳定性更推荐官方 Bot。
