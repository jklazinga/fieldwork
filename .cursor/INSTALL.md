# Installing Fieldwork for Cursor

## Prerequisites

- Git

## Installation

1. **Clone the repository into your home directory:**
   ```bash
   git clone https://github.com/jklazinga/fieldwork.git ~/.cursor/fieldwork
   ```

2. **Copy the agent instructions into your project:**
   ```bash
   cp ~/.cursor/fieldwork/AGENTS.md .cursor/rules/fieldwork.mdc
   ```

   Then add this frontmatter to the top of `.cursor/rules/fieldwork.mdc`:
   ```
   ---
   description: Fieldwork PM skills framework
   alwaysApply: true
   ---
   ```

3. **Restart Cursor** to pick up the new rules.

## Usage

Fieldwork skills activate automatically based on context. To trigger one explicitly, describe what you're doing - e.g. "I want to explore a new feature idea" or "let's write a spec for this."

To run onboarding:
```
Run the fieldwork onboard skill from ~/.cursor/fieldwork/skills/onboard/SKILL.md
```

## Updating

```bash
cd ~/.cursor/fieldwork && git pull
cp ~/.cursor/fieldwork/AGENTS.md .cursor/rules/fieldwork.mdc
```

Then re-add the frontmatter block to the top of `.cursor/rules/fieldwork.mdc`.

## Uninstalling

```bash
rm .cursor/rules/fieldwork.mdc
rm -rf ~/.cursor/fieldwork
```
