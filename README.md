# Lab 10: Linux File Permission Security Scanner

## Learning Objectives

* Use `find` command to locate files with insecure permissions
* Identify world-writable files and directories (security vulnerabilities)
* Detect improperly executable files using permission tests
* Understand real-world file permission security risks

## Prerequisites

* Ubuntu VM (required for this lab)
* Basic Linux command line knowledge
* Understanding of file permissions (numeric and symbolic)
* Completion of Linux File Systems lesson

## Introduction

In production environments, incorrect file permissions create serious security vulnerabilities. A world-writable
configuration file allows any user to modify critical settings. An executable HTML file could be exploited by attackers.
In this lab, you'll build a simple security scanner that finds these dangerous permission issues.

Think of this like a security guard checking all the locks in a building. Your script will scan through directories and
report any "unlocked doors" that could let attackers in.

You'll implement just **2 functions** that use the `find` command to locate insecure files—exactly what security
auditors do in real systems.

## What You'll Implement

Complete **2 TODO tasks** in one file:

* **TODO 1:** Find all world-writable files and directories
* **TODO 2:** Find files that shouldn't be executable

Everything else is provided: test directory creation, output formatting, and main execution.

## Lab File Structure

**security_scanner.sh** - The only file you need

* `setup_test_environment()` - Fully provided (creates test files with bad permissions)
* `find_world_writable()` - TODO 1 (find insecure files/directories)
* `find_executable_non_scripts()` - TODO 2 (find files that shouldn't be executable)
* `main()` - Fully provided (runs everything)

## VM Setup Instructions

**IMPORTANT:** This lab must be completed in your Ubuntu VM.

### Step 1: Clone the Repository

1. Start your Ubuntu VM
2. Open Terminal
3. Clone your lab repository:
```bash
cd ~
git clone <your-repo-url>
cd cus1163-lab10
```

### Step 2: Verify Files

Check that you have the starter file:
```bash
ls -la
# You should see: security_scanner.sh
```

### Step 3: Make Script Executable

```bash
chmod +x security_scanner.sh
```

### Step 4: View the Starter Code

```bash
cat security_scanner.sh
# Or open in your preferred editor:
nano security_scanner.sh
# Or use VSCode
```

## Understanding Permission Security

### What Makes Files Insecure?

| Security Issue           | Permission         | Why It's Dangerous          | Example Attack                         |
|--------------------------|--------------------|-----------------------------|----------------------------------------|
| World-writable file      | `-rw-rw-rw-` (666) | Anyone can modify content   | Attacker changes config to gain access |
| World-writable directory | `drwxrwxrwx` (777) | Anyone can add/delete files | Attacker plants malicious script       |
| Executable HTML/CSS      | `-rwxr-xr-x` (755) | Files run as programs       | Browser exploit executes code          |
| Executable config file   | `-rwxr--r--` (744) | Configs shouldn't run       | Attacker tricks system into running it |

### Find Command Permission Tests

| Find Option  | Meaning                           | Example                   |
|--------------|-----------------------------------|---------------------------|
| `-perm -002` | World-writable (others can write) | Files anyone can modify   |
| `-perm /111` | Any execute bit set               | Files that are executable |
| `-type f`    | Regular files only                | Not directories           |
| `-type d`    | Directories only                  | Not files                 |

## Test Environment Structure

The script automatically creates these files with intentionally bad permissions:

```
~/cus1163-lab10/test_files/
├── web/
│   ├── index.html (777 - INSECURE: world-writable AND executable)
│   ├── style.css (755 - INSECURE: executable)
│   └── script.js (644 - SECURE)
├── config/
│   ├── database.conf (666 - INSECURE: world-writable)
│   ├── api_keys.conf (644 - SECURE)
│   └── settings.conf (755 - INSECURE: executable)
├── scripts/
│   ├── deploy.sh (755 - SECURE: should be executable)
│   └── backup.sh (777 - INSECURE: world-writable)
├── data/
│   ├── users.txt (666 - INSECURE: world-writable)
│   └── logs.txt (640 - SECURE)
└── uploads/ (777 - INSECURE: world-writable directory)
```

Your job: Write `find` commands to locate all the INSECURE items.

## Expected Output

```bash
========================================
File Permission Security Scanner
========================================
Setting up test environment...
Test files created in: /home/student/cus1163-lab10/test_files

========================================
Scanning for INSECURE Files/Directories
========================================

--- World-Writable Files & Directories ---

[FILE] /home/student/cus1163-lab10/test_files/web/index.html (777)
[FILE] /home/student/cus1163-lab10/test_files/config/database.conf (666)
[FILE] /home/student/cus1163-lab10/test_files/scripts/backup.sh (777)
[FILE] /home/student/cus1163-lab10/test_files/data/users.txt (666)
[DIR]  /home/student/cus1163-lab10/test_files/uploads (777)

Found 5 world-writable items

--- Executable Non-Script Files ---

[EXEC] /home/student/cus1163-lab10/test_files/web/index.html (777)
[EXEC] /home/student/cus1163-lab10/test_files/web/style.css (755)
[EXEC] /home/student/cus1163-lab10/test_files/config/settings.conf (755)

Found 3 files that shouldn't be executable

========================================
Security Scan Complete
========================================
Summary:
- World-writable items found: 5
- Improperly executable files found: 3
- Total security issues: 8

⚠️  SECURITY ALERT: 8 permission vulnerabilities detected!
These files need immediate attention.
========================================
```

## Implementation Guide

### TODO 1: Find World-Writable Items

World-writable means **anyone** can modify the file or directory—a critical security flaw.

**Key Concepts:**

```bash
# Find world-writable items (permission bit 002)
find "$TEST_DIR" -perm -002

# Check if item is a file
[ -f "$item" ]

# Check if item is a directory  
[ -d "$item" ]

# Get numeric permissions
stat -c "%a" "$item"  # Returns: 777, 666, etc.

# Increment a counter
((count++))
```

**Step-by-Step Logic:**

1. Use `find` with `-perm -002` to locate world-writable items
2. Loop through each result using **process substitution** (not pipe!)
3. Check if it's a file or directory
4. Get and display permissions
5. Count each item found

**Example Structure:**

```bash
while IFS= read -r item; do
    perms=$(stat -c "%a" "$item")
    
    if [ -f "$item" ]; then
        echo "[FILE] $item ($perms)"
    elif [ -d "$item" ]; then
        echo "[DIR]  $item ($perms)"
    fi
    
    ((count++))
done < <(find "$TEST_DIR" -perm -002)
```

**IMPORTANT:** Use `done < <(find ...)` NOT `find ... | while`. The pipe creates a subshell where the `count` variable won't persist!

### TODO 2: Find Executable Non-Scripts

HTML, CSS, text, and config files should **never** be executable—it's unnecessary and dangerous.

**Key Concepts:**

```bash
# Find executable files matching certain extensions
find "$TEST_DIR" -type f \( -name "*.html" -o -name "*.css" \) -perm /111

# The -perm /111 means "any execute bit set"
# The \( -name ... -o -name ... \) groups multiple OR conditions
```

**Step-by-Step Logic:**

1. Use `find` with multiple `-name` patterns and `-perm /111`
2. Loop through results using **process substitution**
3. Display each executable non-script file
4. Count them

**Example Structure:**

```bash
while IFS= read -r file; do
    perms=$(stat -c "%a" "$file")
    echo "[EXEC] $file ($perms)"
    ((count++))
done < <(find "$TEST_DIR" -type f \( -name "*.html" -o -name "*.css" -o -name "*.txt" -o -name "*.conf" \) -perm /111)
```

## Common Mistakes to Avoid

### 1. Using Pipe Instead of Process Substitution ⚠️ CRITICAL

```bash
# WRONG - Creates subshell, count doesn't persist
find "$TEST_DIR" -perm -002 | while read -r item; do
    ((count++))
done
# count is still 0 here!

# CORRECT - Keeps loop in current shell
while IFS= read -r item; do
    ((count++))
done < <(find "$TEST_DIR" -perm -002)
# count has correct value!
```

### 2. Forgetting to Quote Variables

```bash
# WRONG - Breaks with spaces in paths
find $TEST_DIR -type f

# CORRECT
find "$TEST_DIR" -type f
```

### 3. Missing Parentheses in Find

```bash
# WRONG - OR doesn't group properly
find . -name "*.html" -o -name "*.css" -perm /111

# CORRECT - Parentheses group the OR conditions
find . \( -name "*.html" -o -name "*.css" \) -perm /111
```

### 4. Not Incrementing Counter

```bash
# WRONG - Counter stays at 0
while read -r file; do
    echo "$file"
done

# CORRECT
while read -r file; do
    echo "$file"
    ((count++))
done
```

### 5. Wrong Permission Test

```bash
# WRONG - Different meaning
find . -perm 002   # Exactly 002 (very rare)

# CORRECT
find . -perm -002  # At least world-writable
find . -perm /111  # Any execute bit
```

## Testing Your Script

### Step 1: Run the Script

```bash
cd ~/cus1163-lab10
chmod +x security_scanner.sh
./security_scanner.sh
```

### Step 2: Verify Your Output

Your output should show:

- **5 world-writable items** (4 files + 1 directory)
- **3 executable non-script files** (index.html, style.css, settings.conf)
- **Total: 8 security issues**

### Step 3: Manual Verification

Check the test files manually:

```bash
# List all files with permissions
ls -la ~/cus1163-lab10/test_files/web/
ls -la ~/cus1163-lab10/test_files/config/
ls -la ~/cus1163-lab10/test_files/scripts/
ls -la ~/cus1163-lab10/test_files/data/

# Check directory permissions
ls -ld ~/cus1163-lab10/test_files/uploads/
```

### Step 4: Test Your Find Commands Manually

```bash
# Test TODO 1 command
find ~/cus1163-lab10/test_files -perm -002

# Test TODO 2 command
find ~/cus1163-lab10/test_files -type f \( -name "*.html" -o -name "*.css" -o -name "*.txt" -o -name "*.conf" \) -perm /111
```

## Analysis Questions

After completing the lab, answer these questions:

1. **Security Risk**: Why is a world-writable directory (like `uploads/`) more dangerous than a world-writable file?
   What could an attacker do?
  `` A world-writable directory lets any user create, delete, or replace files inside it. That can lead to code execution, data tampering, or privilege escalation if another service writes/executes from that directory.``

2. **Permission Bits**: Explain what `-perm -002` means versus `-perm /111`. Why do we use different options for
   different searches?
    ``-perm -002 matches anything with the “others write” bit set (world-writable), while -perm /111 matches anything with any execute bit set for user/group/others. We use different tests because one checks “can anyone modify this?” and the other checks “is this runnable as a program?``

3. **Real-World Impact**: If you ran this scanner on a production web server and found 50 world-writable files, what
   immediate actions should you take?
  `` Immediately restrict permissions by removing world-write/execute as appropriate, identify who/what set them through audit logs, and check for compromises like unexpected changes to files. Prioritize configs, web roots, and shared directories, then patch and re-scan. ``

4. **Process Substitution**: Explain why using `find ... | while read` doesn't work for counting, but `while read ... done < <(find ...)` does. What is the difference?
``find ... | while read ... runs the while loop in a subshell, so changes to count disappear when the loop ends. ``


5. **Automation**: How would you schedule this script to run automatically every day at 2 AM and email the results to a
   security team?
   ``Put the script in a cron job to run every day at 2 AM. Have the cron command email (or save) the script’s output so the security team gets the results automatically.``

## What You're Learning

This lab teaches you:

* **Find Command Mastery**: Using permission tests to locate security issues
* **Permission Understanding**: Recognizing dangerous permission patterns
* **Security Mindset**: Thinking like an attacker to find vulnerabilities
* **Shell Scripting Techniques**: Process substitution vs pipes, variable scope
* **Real-World Skills**: Exactly what security auditors do

The key insight: Simple permission mistakes create serious vulnerabilities. Automated scanning prevents breaches before
they happen.

## Execution Commands

```bash
# Navigate to lab directory
cd ~/cus1163-lab10

# Make script executable (only needed once)
chmod +x security_scanner.sh

# Run the script
./security_scanner.sh

# Manually check files
ls -la ~/cus1163-lab10/test_files/web/

# Test find commands directly
find ~/cus1163-lab10/test_files -perm -002
```

## Submission Requirements

Push your completed work to GitHub:

```bash
cd ~/cus1163-lab10
git add security_scanner.sh
git commit -m "Completed Lab 10 - File Permission Security Scanner"
git push origin main
```

Include in your repository:

1. **Completed `security_scanner.sh`** with both TODOs implemented
2. **Screenshots folder** containing:
    - Screenshot showing successful execution with correct counts (5 world-writable, 3 executable)
    - Screenshot of manual `ls -la` verification showing file permissions
3. **ANSWERS.md** file with answers to the 5 analysis questions

## Grading Rubric

| Component                   | Points  | Criteria                                          |
|-----------------------------|---------|---------------------------------------------------|
| TODO 1: World-writable scan | 40      | Correctly finds all 5 world-writable items        |
| TODO 2: Executable scan     | 40      | Correctly finds all 3 improperly executable files |
| Analysis Questions          | 20      | Complete, thoughtful answers                      |
| **Total**                   | **100** |                                                   |

## Troubleshooting

**Script won't execute:**

```bash
chmod +x security_scanner.sh
```

**Wrong count of items (showing 0):**

- Check that you're using `done < <(find ...)` NOT `find ... | while`
- This is the most common issue! Pipes create subshells.

**"syntax error near unexpected token":**

```bash
# Make sure you have space before second 
done < <(find ...)  # CORRECT
done <<(find ...)   # WRONG - no space
```

**Wrong count of items:**

- Verify you're incrementing `count` in the loop
- Make sure you're using `-perm -002` not `-perm 002`

**No output from find:**

```bash
# Verify test files were created
ls -la ~/cus1163-lab10/test_files/
```

**"command not found" error:**

```bash
# Make sure you're in the right directory
cd ~/cus1163-lab10
./security_scanner.sh  # Use ./ to run local script
```

Good luck! Remember: finding insecure permissions is the first step in securing a system.