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

import logging
import re
from pathlib import Path

# ====================
# RENAMING LOGIC
# ====================

def sanitize_filenames(target_folder, dry_run=True):
    """
    Scans a folder and renames files to be 'clean' (no spaces, no special chars).
    """
    target_path = Path(target_folder)
    
    logging.info(f"--- RENAMER STARTED: {target_path} ---")
    
    if not target_path.exists():
        logging.error(f"Target folder does not exist: {target_path}")
        return

    renamed_count = 0
    errors = 0

    # We iterate over the directory
    for current_file in target_path.iterdir():
        if not current_file.is_file():
            continue
            
        # Skip this script and the log file/config
        if current_file.name in ["renamer.py", "__init__.py", "config.json"]:
            continue

        original_name = current_file.name
        new_name = original_name

        # --- RULE 1: Replace spaces with underscores ---
        new_name = new_name.replace(" ", "_")
        
        # --- RULE 2: Remove brackets like (1) or [Copy] ---
        new_name = re.sub(r"[\(\[].*?[\)\]]", "", new_name)
        
        # --- RULE 3: Remove double underscores created by the above ---
        while "__" in new_name:
            new_name = new_name.replace("__", "_")
            
        # --- RULE 4: Strip leading/trailing underscores ---
        new_name = new_name.strip("_")

        # If nothing changed, skip it
        if new_name == original_name:
            continue
            
        # Construct full path
        destination = current_file.parent / new_name
        
        try:
            if dry_run:
                logging.info(f"[DRY RUN] Would rename: '{original_name}' -> '{new_name}'")
            else:
                # Check if destination already exists to prevent overwriting
                if destination.exists():
                    logging.warning(f"Skipping '{original_name}': Target '{new_name}' already exists.")
                    errors += 1
                    continue
                    
                current_file.rename(destination)
                logging.info(f"[RENAMED] '{original_name}' -> '{new_name}'")
                renamed_count += 1
                
        except Exception as e:
            logging.error(f"Failed to rename {original_name}: {e}")
            errors += 1

    logging.info(f"--- RENAMER FINISHED: Renamed {renamed_count} files. Errors: {errors} ---")


# ====================
# STANDALONE MODE
# ====================
if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO, format='%(message)s')
    
    BASE_DIR = Path(__file__).parent
    TEST_TARGET = BASE_DIR / "test_files"
    
    # Create a messy file to test
    TEST_TARGET.mkdir(exist_ok=True)
    messy_file = TEST_TARGET / "My  Messy File (1).txt"
    messy_file.touch()
    
    print("Running in STANDALONE mode...")
    # Set dry_run=False if you want to actually see the file change name
    sanitize_filenames(TEST_TARGET, dry_run=True)


#### Author ####

Zain Barkatali
Barkatali Technology
Automation & IT systems for small businesses
