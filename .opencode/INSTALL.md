# Installing Design Patterns for OpenCode

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/VoDaiLocz/Design-Patterns.git ~/.config/opencode/design-patterns
   ```

2. **Symlink Skills:**
   ```bash
   mkdir -p ~/.config/opencode/skills
   ln -s ~/.config/opencode/design-patterns/skills ~/.config/opencode/skills/design-patterns
   ```

3. **Restart OpenCode.**

## Usage

```
/design-patterns src/api/users
```

## Updating

```bash
cd ~/.config/opencode/design-patterns && git pull
```

## Getting Help

- Issues: https://github.com/VoDaiLocz/Design-Patterns/issues
