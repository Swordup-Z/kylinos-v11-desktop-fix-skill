# kylinos-desktop-v11-skill

用于沉淀和复用 KylinOS Desktop V11 桌面系统问题的诊断、修复与验证经验，覆盖 UKUI、KARE/Kaiming、TUN、开机自启动、维护模式、磐石架构、系统服务、分区挂载和 overlay 等场景。

This repository provides a reusable skill and knowledge base for diagnosing and fixing KylinOS Desktop V11 system issues.

## Usage

Use `SKILL.md` as the entry point:

```bash
sed -n '1,160p' "$HOME/kylinos-desktop-v11-skill/SKILL.md"
```

For AI coding tools, configure a user-level global prompt to route KylinOS Desktop V11 system issues to this skill. See:

```text
references/agent-global-prompts.md
```

The intended loading flow is progressive:

```text
SKILL.md -> relevant references/*.md -> diagnosis -> repair -> verification -> record reusable findings
```

## Safety

System-level repairs on KylinOS Desktop V11 may require maintenance mode. Before modifying `/usr`, `/etc`, `/opt`, system packages, services, device nodes, partitions, or KSaf policy, check:

```bash
mm-cli -s
```

Only proceed with system-level changes after confirming maintenance mode.

## License

MIT
