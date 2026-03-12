---
AIGC:
    ContentProducer: Minimax Agent AI
    ContentPropagator: Minimax Agent AI
    Label: AIGC
    ProduceID: ce6c9b9ba6ffcae5d3d69b18fe408b8f
    PropagateID: ce6c9b9ba6ffcae5d3d69b18fe408b8f
    ReservedCode1: 30450221008c10ee9c5fbd3d41a3d617e9deab3ba123683ae6aaea0a358898e25c33a47ac7022048abbe3b0d8654e4eda1db4cfadc9dada0ed48457a2e8791b0924970efeeabc4
    ReservedCode2: 3045022100f9347db2e95f4d7cdf1599717055b98e5d671d3de3e70aabbfe55f0d6b21b47c022046b928cc16c5b21bc36d7af7397f8aec8317af72c15d4c22b8d3eeedab4a1e93
---

# TOOLS.md - Local Notes

Skills define _how_ tools work. This file is for _your_ specifics — the stuff that's unique to your setup.

## What Goes Here

Things like:

- Camera names and locations
- SSH hosts and aliases
- Preferred voices for TTS
- Speaker/room names
- Device nicknames
- Anything environment-specific

## Examples

```markdown
### Cameras

- living-room → Main area, 180° wide angle
- front-door → Entrance, motion-triggered

### SSH

- home-server → 192.168.1.100, user: admin

### TTS

- Preferred voice: "Nova" (warm, slightly British)
- Default speaker: Kitchen HomePod
```

## Why Separate?

Skills are shared. Your setup is yours. Keeping them apart means you can update skills without losing your notes, and share skills without leaking your infrastructure.

---

### GitHub

- **仓库**: github-project-push (kkpian5599-gif)
- **SSH Key**: ~/.ssh/id_ed25519 (Ed25519)
- **用途**: 定时推送GitHub trending项目
- **配置日期**: 2026-03-12

### 使用说明

1. 首次使用需要将公钥添加到GitHub → Settings → SSH and GPG keys
2. 确保远程仓库使用SSH URL: `git@github.com:kkpian5599-gif/github-project-push.git`
