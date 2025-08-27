# Dmenu FlexiPatch Production Branch Management

## Updating from Upstream

### Prerequisites
Ensure the upstream remote is configured:
```bash
git remote add upstream https://github.com/bakkeby/dmenu-flexipatch.git
```

### Step 1: Sync Master with Upstream
First, update the pristine `master` branch:
```bash
git checkout master
git fetch upstream master
git rebase --rebase-merge upstream/master
git push origin master
```

**Note**: There should be no conflicts here since `master` must remain pristine and unchanged.

### Step 2: Rebase Production Branch

**Warning**: This will likely produce conflicts that need to be resolved.

Rebase `prod` branch against the updated `master`:
```bash
git checkout prod
git rebase --rebase-merges master
```

**Note**: The `--rebase-merges` option preserves merge commits and ensures you don't have to resolve conflicts that were already resolved in previous merges.

#### Handling Conflicts
If merge conflicts occur:

1. **Resolve conflicts** in the affected files (use `git status` to see conflicted files)
   ```bash
   git status
   # Edit conflicted files to resolve conflicts
   # Optionally use mergetool:
   git mergetool
   ```

2. **Stage resolved files** and continue the rebase:
   ```bash
   git add resolved_file.c
   git rebase --continue
   ```

3. **If things go wrong**, you can always abort:
   ```bash
   git rebase --abort
   ```

### Step 3: Build and Test

After rebasing, rebuild and test dmenu:
```bash
./install.sh
```

This script will:
- Clean the build
- Remove existing `patches.h` and `config.h`
- Rebuild dmenu
- Install it system-wide (requires sudo)

### Step 4: Push Changes

Once everything is working:
```bash
git push --force-with-lease origin prod
```

**Note**: Force push is required after rebasing. The `--force-with-lease` option is safer than `--force` as it ensures you don't overwrite any remote changes you haven't seen.

## Quick Reference

### Check Current Branch Status
```bash
git status
git log --oneline --graph --decorate -10
```

### View Differences
```bash
git diff master..prod  # See changes in prod vs master
```

### Emergency Rollback
If an update breaks something critical:
```bash
git checkout prod
git reset --hard origin/prod  # Reset to last known good state
```

## Testing Dmenu

### Quick Test Commands
```bash
# Test basic functionality
echo -e "Option 1\nOption 2\nOption 3" | dmenu

# Test with custom prompt
echo -e "Firefox\nChrome\nTerminal" | dmenu -p "Launch:"

# Test grid layout (if GRID_PATCH enabled)
ls /usr/bin | dmenu -g 3 -l 10

# Test fuzzy matching (if FUZZYMATCH_PATCH enabled) 
echo -e "firefox\nfirefox-developer\nchromium" | dmenu -F

# Test with colors and positioning
echo -e "Red\nGreen\nBlue" | dmenu -nb '#1e1e2e' -nf '#cdd6f4' -sb '#89b4fa' -sf '#1e1e2e'
```

### Integration Testing
```bash
# Test with dmenu_run (application launcher)
dmenu_run

# Test with dmenu_path (show available executables)
dmenu_path | dmenu
```

## Important Notes

1. **Never modify `master`** - It must remain identical to upstream/master
2. **Always test** after rebasing before pushing
3. **Keep backups** of your `config.h` and `patches.h` before updating
4. **Document conflicts** - Keep notes on recurring conflict resolutions for future updates
5. **Test all enabled patches** - Ensure your custom patch combinations still work after updates

## Common Conflict Areas

When rebasing, conflicts typically occur in:
- `config.def.h` - If you've customized keybindings or colors
- `patches.def.h` - If upstream added new patches or modified existing ones
- `dmenu.c` - If patches you use have been updated upstream
- `config.mk` - If build dependencies changed

## Patch Configuration Backup

Before updating, backup your configurations:
```bash
cp config.h config.h.backup
cp patches.h patches.h.backup
```

After successful update, compare and restore customizations as needed.