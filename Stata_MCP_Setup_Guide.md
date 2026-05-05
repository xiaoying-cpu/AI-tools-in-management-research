# Complete Guide: Installing and Configuring Stata MCP with Claude Code

## Overview

This guide walks you through successfully installing and configuring **Stata MCP (Model Context Protocol)** with **Claude Code** on Windows. This setup allows Claude Code to directly execute Stata do-files and return results.

**Difficulty Level:** Intermediate  
**Time Required:** 15-30 minutes  
**Platform:** Windows 11 (PowerShell)

---

## Prerequisites

Before starting, ensure you have:

- ✅ **Claude Code installed** (v2.1.79 or later)
- ✅ **Stata installed** (Stata 17 or 18 recommended)
- ✅ **PowerShell** (Windows PowerShell or PowerShell 7+)
- ✅ **UV** (Python package manager) — installed automatically with Claude Code
- ✅ **Claude Pro or Max subscription** (required for Claude Code)

**Check your versions:**

```powershell
claude --version
uv --version
```

---

## Step 1: Install the Stata MCP VS Code Extension

The Stata MCP extension provides the integration layer between Claude Code and your Stata installation.

### Installation Steps:

1. **Open VS Code**
2. **Go to Extensions** (`Ctrl + Shift + X`)
3. **Search for:** `stata-mcp` or `deepecon.stata-mcp`
4. **Click Install**
5. **Wait for setup to complete** — The extension will:
   - Check for UV and Python
   - Create a Python virtual environment
   - Install dependencies
   - Start the MCP server

### Verify Installation:

Check the **Output panel** (bottom of VS Code) for messages like:

```
Setup completed successfully!
Python environment setup successfully
Starting MCP server with configured Python environment...
```

If you see these messages, the extension is ready! ✅

---

## Step 2: Connect Claude Code to Stata MCP

Now you need to register the Stata MCP server with Claude Code.

### In PowerShell, run:

```powershell
claude mcp add stata-mcp --scope user -- "C:\Users\YourUsername\.local\bin\uvx.exe" stata-mcp
```

**Replace `YourUsername`** with your actual Windows username (e.g., `C:\Users\User\.local\bin\uvx.exe`).

### Expected Output:

```
Added stdio MCP server stata-mcp with command: C:\Users\YourUsername\.local\bin\uvx.exe stata-mcp to user config
File modified: C:\Users\YourUsername\.claude.json
```

✅ This creates/updates the `.claude.json` file with the Stata MCP configuration.

---

## Step 3: Restart Claude Code

You must fully restart Claude Code for the configuration to take effect.

### Steps:

1. **Close any running Claude Code sessions** (type `exit` in PowerShell)
2. **Wait 2-3 seconds**
3. **Reopen Claude Code** by running:

```powershell
claude
```

---

## Step 4: Verify the Connection

Check if Stata MCP is properly connected.

### In Claude Code, run:

```
/mcp
```

### Expected Output:

You should see a menu showing:

```
Manage MCP servers
5 servers
  User MCPs (C:\Users\YourUsername\.claude.json)
> stata-mcp · √ connected · 4 tools
  claude.ai
  claude.ai Gmail · √ connected · 10 tools
  ...
```

✅ **If you see `√ connected · 4 tools`**, your setup is successful!

The 4 tools are:
1. `stata_do` — Execute Stata do-files
2. `get_data_info` — Get data information
3. `read_log` — Read Stata log files
4. `ado_package_install` — Install Stata packages

---

## Step 5: Test Your Setup

### Basic Test:

In Claude Code, ask:

```
Load the auto dataset in Stata and show summary statistics
```

### Expected Result:

Claude Code should:
1. Write a do-file with Stata commands
2. Execute it through Stata MCP
3. Read the output log
4. Display a formatted summary statistics table

**Example Output:**

```
┌──────────────┬─────┬──────────┬───────────┬───────┬────────┐
│   Variable   │ Obs │   Mean   │ Std. Dev. │  Min  │  Max   │
├──────────────┼─────┼──────────┼───────────┼───────┼────────┤
│ price        │ 74  │ 6,165.26 │ 2,949.50  │ 3,291 │ 15,906 │
│ mpg          │ 74  │ 21.30    │ 5.79      │ 12    │ 41     │
└──────────────┴─────┴──────────┴───────────┴───────┴────────┘
```

✅ **If you see this table, your setup is complete!**

---

## Troubleshooting

### Issue 1: Stata MCP Shows `× failed` Instead of `√ connected`

**Cause:** Claude Code can't find the UV executable in its PATH.

**Solution:**

1. **Find UV's full path:**
   ```powershell
   Get-Command uvx | Select-Object -ExpandProperty Source
   ```

2. **Remove the old configuration:**
   ```powershell
   claude mcp remove stata-mcp
   ```

3. **Re-add with the full path** (replace with your actual path):
   ```powershell
   claude mcp add stata-mcp --scope user -- "C:\Users\YourUsername\.local\bin\uvx.exe" stata-mcp
   ```

4. **Close and reopen Claude Code:**
   ```powershell
   exit
   claude
   ```

5. **Verify with `/mcp`** — should now show `√ connected`

---

### Issue 2: "No suitable shell found" Error

**Cause:** Claude Code requires a POSIX shell environment.

**Solution:**

Install **Git Bash** or use **Windows Terminal** instead of plain PowerShell. Most users with Python and UV installed already have this.

---

### Issue 3: Stata Not Detected

**Cause:** Claude Code can't find your Stata installation.

**Solution:**

1. **Verify Stata is installed:**
   ```powershell
   Get-ChildItem "C:\Program Files\Stata*"
   ```

2. **Check if Stata is in PATH:**
   ```powershell
   Get-Command stata
   ```

3. **If not found**, manually add Stata to your system PATH:
   - Open Windows Settings → System → Environment Variables
   - Add Stata's bin directory (e.g., `C:\Program Files\Stata18`) to PATH
   - Restart PowerShell

---

### Issue 4: MCP Server Starts But Claude Code Can't Communicate

**Cause:** Configuration mismatch or stale Claude Code session.

**Solution:**

1. **Exit Claude Code completely:**
   ```powershell
   exit
   ```

2. **Check the configuration:**
   ```powershell
   cat C:\Users\YourUsername\.claude.json | findstr stata-mcp
   ```

3. **Restart Claude Code:**
   ```powershell
   claude
   ```

4. **Test again with `/mcp`**

---

## Common Questions

### Q: Do I need to keep VS Code open?

**A:** No. The Stata MCP extension runs in the background once installed. You only need VS Code open if you want to use the extension features directly. Claude Code in PowerShell works independently.

### Q: Can I use this with Stata 17?

**A:** Yes! The setup works with both Stata 17 and 18. Claude Code will automatically detect whichever version is installed (or use the newer one if both are present).

### Q: How many tokens does this use?

**A:** The Stata MCP is configured in "compact" mode, which filters redundant output and uses fewer tokens than the standard method. For a simple summary statistics command, expect 10-30 tokens depending on output size.

### Q: Can I run my own do-files?

**A:** Absolutely! You can ask Claude Code to:
- Load and execute your existing `.do` files
- Create new do-files from prompts
- Debug and fix errors in your code
- Run analysis on your data

Example:
```
"Execute my do-file at C:\path\to\my_analysis.do and show me the results"
```

### Q: What if I have multiple Stata installations?

**A:** Claude Code will use the newest version by default. If you need to force a specific version, you can modify the Stata MCP configuration, but this is advanced and usually not necessary.

---

## Next Steps

Once your setup is complete, you can:

1. **Run analysis:** Ask Claude Code to load your data and run statistical analyses
2. **Create visualizations:** Generate plots and charts
3. **Debug code:** Have Claude Code fix Stata syntax errors
4. **Batch processing:** Run multiple do-files and combine results
5. **Learn Stata:** Ask Claude Code for explanations and best practices

### Example Commands:

```
# Run a regression
"Run a multiple regression with price as dependent variable and mpg, weight, and foreign as independent variables"

# Create a plot
"Generate a scatter plot of price vs mpg with a regression line"

# Process your own data
"Load my dataset from C:\data\mydata.dta and create summary statistics by group"

# Debug code
"I have this do-file with errors. Can you fix it and run it? [paste code]"
```

---

## Resources

- **Claude Code Documentation:** https://docs.anthropic.com/en/docs/claude-code/overview
- **Stata MCP GitHub:** https://github.com/hanlulong/stata-mcp
- **Claude Code CLI Reference:** `claude --help`

---

## Summary of Commands

Here's a quick reference of all commands used:

```powershell
# Check versions
claude --version
uv --version

# Add Stata MCP
claude mcp add stata-mcp --scope user -- "C:\Users\YourUsername\.local\bin\uvx.exe" stata-mcp

# Start Claude Code
claude

# Check MCP connection (inside Claude Code)
/mcp

# Remove Stata MCP (if needed)
claude mcp remove stata-mcp

# Debug MCP issues (inside Claude Code)
/exit
claude --debug
```

---

## Troubleshooting Checklist

If your setup isn't working, go through this checklist:

- [ ] Claude Code version 2.1.79 or later?
- [ ] Stata MCP VS Code extension installed?
- [ ] UV installed and working? (`uv --version`)
- [ ] Stata MCP added to Claude Code? (`claude mcp list`)
- [ ] Claude Code fully restarted after setup?
- [ ] `/mcp` shows `√ connected` for stata-mcp?
- [ ] Stata installed and in PATH?
- [ ] Using PowerShell (not CMD)?

If all boxes are checked and it's still not working, try the **nuclear option:**

```powershell
# Remove completely
claude mcp remove stata-mcp

# Reinstall with fresh configuration
claude mcp add stata-mcp --scope user -- "C:\Users\YourUsername\.local\bin\uvx.exe" stata-mcp

# Fully close and reopen
exit
claude
```

---

## Credits & Notes

This guide is based on the actual setup process on Windows 11 with:
- Claude Code v2.1.128
- Stata 17 & 18
- UV 0.11.8
- Stata MCP v1.16.2

The main challenge in this setup is ensuring Claude Code can find the UV executable. The solution is using the full path instead of relying on system PATH variables.

---

**Last Updated:** May 2026  
**Status:** ✅ Tested and Working
