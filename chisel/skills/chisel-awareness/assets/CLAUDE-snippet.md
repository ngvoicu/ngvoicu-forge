## Chisel (MANDATORY)

Fast surgical code editor. **NEVER manually rename or move files — always use chisel to ensure all references are updated.**

### When to Use Chisel

| Situation | Command |
|-----------|---------|
| Rename a file, variable, function, or class | `/chisel rename <old> to <new>` |
| Move a file or directory | `/chisel move <source> to <destination>` |
| Find-and-replace text across files | `/chisel replace <old> with <new>` |
| Small CSS/HTML/config tweaks | `/chisel edit <description>` |
| Any rename, move, or targeted edit | `/chisel <natural language description>` |

### Rules

- **NEVER rename or move files manually** (with `mv`, manual delete+create, or IDE refactoring) — always use `/chisel` to ensure all imports, references, and config files are updated
- **NEVER do find-and-replace with sed/awk** — use `/chisel replace` which handles word boundaries, scope analysis, and verification
- **ALWAYS verify** that chisel found and updated all references — check the summary it produces
- **Use `/chisel rename`** for any entity rename: files, directories, variables, functions, classes, components, modules

### Commands

```
/chisel                                # Show available operations
/chisel rename <old> to <new>          # Rename + update all references
/chisel move <source> to <dest>        # Move + update all import paths
/chisel replace <old> with <new>       # Find-and-replace with scope analysis
/chisel edit <description>             # Small targeted code change
/chisel <natural language>             # Auto-detect operation
```

### Integration with Other Plugins

**With specsmith:** When implementing a spec that requires file renames or structural changes, use chisel to ensure all references stay intact.

**With runebook:** After chisel operations, runebook entries referencing affected files are updated automatically. If `.runebook/` exists, chisel changes trigger runebook awareness.

**With grimoire:** For large renames, use `/grimoire codebase` first to understand the full scope of affected files before running chisel.

**With uispec:** When renaming endpoints or API handlers, chisel updates both source files and corresponding uispec endpoint specs.

### Notes

- Chisel uses `git mv` for file operations in git repositories to preserve file history
- For operations affecting >5 files, chisel previews all changes and asks for confirmation
- Chisel works on any language — JavaScript, TypeScript, Python, Go, Rust, Java, CSS, HTML, YAML, JSON, and more
- After every operation, chisel verifies no orphaned references remain
