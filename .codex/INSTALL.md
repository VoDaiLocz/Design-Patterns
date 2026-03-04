# Installing Design Patterns for Codex

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/VoDaiLocz/Design-Patterns.git ~/.codex/design-patterns
   ```

2. **Create the skills symlink:**
   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/design-patterns/skills ~/.agents/skills/design-patterns
   ```

   **Windows (PowerShell):**
   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\design-patterns" "$env:USERPROFILE\.codex\design-patterns\skills"
   ```

3. **Restart Codex** to discover the skill.

## Usage

```
/design-patterns src/api/users
```

## Updating

```bash
cd ~/.codex/design-patterns && git pull
```

## Uninstalling

```bash
rm ~/.agents/skills/design-patterns
rm -rf ~/.codex/design-patterns
```
