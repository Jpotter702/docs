# Batcat Quick Reference

## Basic Usage
- `batcat file.txt` - Basic syntax highlighting
- `batcat --style=plain file.txt` - No decorations
- `batcat --style=numbers,grid file.txt` - Show line numbers and grid
- `batcat --style=header,grid file.txt` - Show header and grid

## Piping Examples
- `command | batcat --plain --paging=never` - Highlight piped content
- `echo "some code" | batcat --language=python` - Force language
- `diff file1 file2 | batcat --language=diff` - Highlight diff output

## Useful Options
- `--style=plain,header,grid,numbers` - Combined styles
- `--language=xyz` - Force specific language
- `--paging=never` - Disable paging
- `--theme=ansi` - Use ANSI theme (works in all terminals)

## Tips
- Create alias: `alias bat=batcat`
- Set theme: `export BAT_THEME="Dracula"`
- Set style: `export BAT_STYLE="plain"`
