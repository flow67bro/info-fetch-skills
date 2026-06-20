# info-fetch-skills

信息获取类 agent skills —— 帮你从互联网上高效获取、筛选、对比信息。

## 已有 Skills

| Skill | 用途 |
|-------|------|
| [douban-best-edition-webaccess](./douban-best-edition-webaccess/) | 豆瓣找最佳版本 → 自动查微信读书能否直接看（译本对比、出版社对比、繁简对比、微信读书可用性） |

## 安装

### 一键安装单个 skill

```bash
# 豆瓣查最佳版本
mkdir -p ~/.agents/skills/douban-best-edition-webaccess && \
  curl -fsSL https://raw.githubusercontent.com/flow67bro/info-fetch-skills/main/douban-best-edition-webaccess/SKILL.md \
  -o ~/.agents/skills/douban-best-edition-webaccess/SKILL.md
```

### 克隆全部 skill

```bash
git clone https://github.com/flow67bro/info-fetch-skills.git /tmp/info-fetch-skills && \
  cp -r /tmp/info-fetch-skills/*/ ~/.agents/skills/ && \
  rm -rf /tmp/info-fetch-skills
```

### 安装指定版本

```bash
# 安装 v1.0.0（仅豆瓣对比，无微信读书集成）
curl -fsSL https://raw.githubusercontent.com/flow67bro/info-fetch-skills/v1.0.0/douban-best-edition-webaccess/SKILL.md \
  -o ~/.agents/skills/douban-best-edition-webaccess/SKILL.md
```

所有历史版本见 [Releases](https://github.com/flow67bro/info-fetch-skills/releases)。

安装后 agent 会自动识别 `~/.agents/skills/` 下的 skill。

## 依赖

`douban-best-edition-webaccess` 完整功能需要以下 skill：

| Skill | 必需？ | 用途 | 安装 |
|-------|--------|------|------|
| [web-access](https://github.com/eze-is/web-access) | 建议 | Jina 抓取失败时的浏览器后备 | `git clone https://github.com/eze-is/web-access.git /tmp/wa && cp -r /tmp/wa/* ~/.agents/skills/web-access/` |
| [weread-skills](https://github.com/flow67bro/weread-skills) | 可选 | Phase 5 微信读书上架匹配 | 见该仓库 README |

> 缺少 `weread-skills` 不影响豆瓣评分对比的核心功能，仅跳过微信读书可用性检查。

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| [v1.1.0](https://github.com/flow67bro/info-fetch-skills/releases/tag/v1.1.0) | 2026-06-19 | 新增微信读书可用性检查（Phase 5），内容评价输出前置 |
| [v1.0.0](https://github.com/flow67bro/info-fetch-skills/releases/tag/v1.0.0) | 2026-06-18 | 初始版本：豆瓣版本对比 |

## 贡献

如果你有信息获取相关的实用 skill，欢迎 PR。
