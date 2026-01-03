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

The code for this should be as follows:

from pathlib import Path
from datetime import datetime
import re

# =======================
# CONFIG
# =======================
BASE_DIR = Path(__file__).parent
TARGET_FOLDER = BASE_DIR / "test_files"
LOG_FILE = BASE_DIR / "rename_log.txt"

DRY_RUN = True

# Rename rules
LOWERCASE = True
REPLACE_SPACES_WITH = "_"   # set to None to leave spaces
REMOVE_SPECIAL_CHARS = True # keeps letters, numbers, underscores, hyphens
ADD_PREFIX = ""             # e.g. "client_"
ADD_SUFFIX = ""             # e.g. "_archived"

# Optional numbering
ADD_NUMBERING = False
NUMBER_START = 1
NUMBER_PADDING = 3          # 001, 002, 003
# =======================


def log_line(action: str, message: str) -> None:
    ts = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    with open(LOG_FILE, "a", encoding="utf-8") as f:
        f.write(f"{ts} | {action:<7} | {message}\n")


def clean_name(stem: str) -> str:
    name = stem

    if LOWERCASE:
        name = name.lower()

    if REPLACE_SPACES_WITH is not None:
        name = name.replace(" ", REPLACE_SPACES_WITH)

    if REMOVE_SPECIAL_CHARS:
        # Keep letters/numbers/_/-
        name = re.sub(r"[^a-zA-Z0-9_-]", "", name)

    # Avoid empty names
    if not name.strip():
        name = "file"

    return name


def unique_path(path: Path) -> Path:
    """If target name exists, add _1, _2, etc to avoid overwriting."""
    if not path.exists():
        return path

    base = path.stem
    ext = path.suffix
    parent = path.parent

    i = 1
    while True:
        candidate = parent / f"{base}_{i}{ext}"
        if not candidate.exists():
            return candidate
        i += 1


def main() -> None:
    if not TARGET_FOLDER.exists():
        print(f"[ERROR] Target folder not found: {TARGET_FOLDER}")
        log_line("ERROR", f"Target folder not found: {TARGET_FOLDER}")
        return

    log_line("START", f"DRY_RUN={DRY_RUN} TARGET={TARGET_FOLDER}")

    files = [p for p in TARGET_FOLDER.iterdir() if p.is_file()]
    if not files:
        print("No files found to rename.")
        log_line("DONE", "No files found")
        return

    counter = NUMBER_START

    for file in files:
        original = file.name
        ext = file.suffix

        new_stem = clean_name(file.stem)
        new_stem = f"{ADD_PREFIX}{new_stem}{ADD_SUFFIX}"

        if ADD_NUMBERING:
            new_stem = f"{new_stem}_{str(counter).zfill(NUMBER_PADDING)}"
            counter += 1

        new_path = file.with_name(new_stem + ext)
        new_path = unique_path(new_path)

        if new_path.name == original:
            continue

        if DRY_RUN:
            print(f"[DRY RUN] {original}  ->  {new_path.name}")
            log_line("DRYRUN", f"{original} -> {new_path.name}")
        else:
            file.rename(new_path)
            print(f"[RENAMED] {original}  ->  {new_path.name}")
            log_line("RENAMED", f"{original} -> {new_path.name}")

    log_line("DONE", "Completed rename run")
    print(f"Done. Log written to: {LOG_FILE}")


if __name__ == "__main__":
    main()


#### Author ####

Zain Barkatali
Barkatali Technology
Automation & IT systems for small businesses
