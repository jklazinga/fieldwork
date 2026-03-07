# Installing Fieldwork for Codex

## Prerequisites

- Git

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/jklazinga/fieldwork.git ~/.codex/fieldwork
   ```

2. **Create the skills symlink:**
   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/fieldwork/skills ~/.agents/skills/fieldwork
   ```

3. **Add instructions to your AGENTS.md:**

   Add this to `~/.codex/AGENTS.md`:
   ```
   Fieldwork skills are available in ~/.agents/skills/fieldwork/. Read CLAUDE.md in the fieldwork repo for full instructions.
   ```

4. **Restart Codex** to discover the skills.

## Updating

```bash
cd ~/.codex/fieldwork && git pull
```

## Uninstalling

```bash
rm ~/.agents/skills/fieldwork
rm -rf ~/.codex/fieldwork
```
