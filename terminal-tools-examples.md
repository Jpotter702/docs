# Useful Tool Combinations

## File Viewing & Editing Workflows

### View and Edit Markdown
```bash
# Preview markdown while editing
glow -p file.md        # In one terminal
micro file.md          # In another terminal

# Quick edit-preview cycle
micro file.md && glow file.md
```

### Advanced File Management
```bash
# Find and edit files from ranger
ranger                 # Navigate to file
'e' key               # Edit in micro
'S' key               # Open shell in current dir

# Search and edit workflow
find . -name "*.md" | bat --style=numbers
find . -name "*.md" | xargs micro  # Edit found files
```

### Text Processing Pipelines
```bash
# Process and highlight output
ls -la | bat --style=plain --language=bash
EOFger && glow .                   # Navigate then previewss
