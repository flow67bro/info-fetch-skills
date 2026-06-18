# info-fetch-skills

信息获取类 agent skills —— 帮你从互联网上高效获取、筛选、对比信息。

## 已有 Skills

| Skill | 用途 |
|-------|------|
| [douban-best-edition-webaccess](./douban-best-edition-webaccess/) | 通过豆瓣评分和书评，帮你找到任意一本书的最佳版本（译本对比、出版社对比、繁简对比） |

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

安装后 agent 会自动识别 `~/.agents/skills/` 下的 skill。

## 贡献

如果你有信息获取相关的实用 skill，欢迎 PR。
