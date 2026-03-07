# Installing Fieldwork for OpenCode

## Prerequisites

- [OpenCode](https://opencode.ai) installed
- Git

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/jklazinga/fieldwork.git ~/.config/opencode/fieldwork
   ```

2. **Add Fieldwork to your global OpenCode instructions:**

   Append this to `~/.config/opencode/AGENTS.md` (create it if it doesn't exist):
   ```
   Fieldwork skills are available in ~/.config/opencode/fieldwork/skills/. Read ~/.config/opencode/fieldwork/CLAUDE.md for full instructions. Before any feature work, planning, or go-to-market activity, check whether a relevant Fieldwork skill applies.
   ```

3. **Restart OpenCode.**

## Usage

Fieldwork skills activate automatically based on context. To trigger one explicitly, describe what you're doing - e.g. "I want to explore a new feature idea" or "let's write a spec for this."

To run onboarding:
```
Run the fieldwork onboard skill from ~/.config/opencode/fieldwork/skills/onboard/SKILL.md
```

## Updating

```bash
cd ~/.config/opencode/fieldwork && git pull
```

## Uninstalling

```bash
rm -rf ~/.config/opencode/fieldwork
```

Then remove the Fieldwork lines from `~/.config/opencode/AGENTS.md`.
