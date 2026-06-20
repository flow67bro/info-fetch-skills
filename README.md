# info-fetch-skills

信息获取类 agent skills —— 帮你从互联网上高效获取、筛选、对比信息。

## 已有 Skills

| Skill | 用途 |
|-------|------|
| [douban-best-edition](./douban-best-edition/) | 豆瓣找最佳版本 → 自动查微信读书能否直接看（译本对比、出版社对比、繁简对比、微信读书可用性） |

## 安装

### 一键安装单个 skill

```bash
# 豆瓣查最佳版本
mkdir -p ~/.agents/skills/douban-best-edition && \
  curl -fsSL https://raw.githubusercontent.com/flow67bro/info-fetch-skills/main/douban-best-edition/SKILL.md \
  -o ~/.agents/skills/douban-best-edition/SKILL.md
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
curl -fsSL https://raw.githubusercontent.com/flow67bro/info-fetch-skills/v1.0.0/douban-best-edition/SKILL.md \
  -o ~/.agents/skills/douban-best-edition/SKILL.md
```

所有历史版本见 [Releases](https://github.com/flow67bro/info-fetch-skills/releases)。

安装后 agent 会自动识别 `~/.agents/skills/` 下的 skill。

## 依赖

`douban-best-edition` 的核心功能（豆瓣版本对比）零依赖，仅 curl + Jina 即可运行。

| Skill | 必需？ | 用途 |
|-------|--------|------|
| [weread-skills](https://github.com/flow67bro/weread-skills) | 可选 | Phase 5 微信读书上架匹配 + `weread://` 直达链接 |

> 缺少 `weread-skills` 时，Phase 5 自动跳过，豆瓣评分对比完全不受影响。

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| [v1.1.0](https://github.com/flow67bro/info-fetch-skills/releases/tag/v1.1.0) | 2026-06-19 | 新增微信读书可用性检查（Phase 5），内容评价输出前置 |
| [v1.0.0](https://github.com/flow67bro/info-fetch-skills/releases/tag/v1.0.0) | 2026-06-18 | 初始版本：豆瓣版本对比 |

## 贡献

如果你有信息获取相关的实用 skill，欢迎 PR。
