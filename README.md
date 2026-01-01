# Bulk-File-Renamer
Safe bulk file renaming tool with dry-run and logging

# Bulk File Renamer

A safe Python tool for renaming files in bulk using configurable rules.

## Features
- Dry-run mode (preview changes before applying)
- Collision-safe renaming (no overwrites)
- Optional numbering
- Full audit log of changes

## Usage
1. Place files inside `test_files`
2. Adjust rules in `renamer.py`
3. Run in dry-run mode first:
   ```Run the command in PowerShell below:
   python renamer.py
Before I do that though - I need to change the file path in PowerShell so that my new command under pythno renamer.py takes effect, not python cleaner.py
