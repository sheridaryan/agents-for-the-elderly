# Git Command Reference
*Created April 2026 — Agents for the Elderly Project*

---

## Setup Commands
Use these once when setting up Git on a new machine.

| Command | What it does |
|---|---|
| `git config --global user.name "Name"` | Sets your name for all commits |
| `git config --global user.email "email"` | Sets your email for all commits |
| `git config --global core.editor "nano"` | Sets your preferred text editor |
| `git config --global --list` | Shows all your current Git settings |
| `git --version` | Confirms Git is installed and shows the version |

---

## Starting a Repository
Use these once when connecting a folder to GitHub for the first time.

| Command | What it does |
|---|---|
| `git init` | Turns a folder into a Git repository |
| `git remote add origin [url]` | Connects your local folder to GitHub |
| `git push -u origin master` | First push only — establishes the connection |

---

## The Everyday Cycle
These are the commands you'll use every working session.

| Command | What it does |
|---|---|
| `git add .` | Stages all changed files ready to commit |
| `git commit -m "message"` | Saves a snapshot with a description |
| `git push` | Sends your commits up to GitHub |

---

## Checking Your Work
Use these to see what's changed before committing.

| Command | What it does |
|---|---|
| `git status` | Shows what files have changed |
| `git diff` | Shows exactly what changed inside files |

---

## Navigation Commands
Use these to move around in Git Bash.

| Command | What it does |
|---|---|
| `pwd` | Shows which folder you're currently in |
| `ls` | Lists all files in the current folder |
| `cd ~/Documents` | Navigates to your Documents folder |
| `mkdir foldername` | Creates a new folder |
| `cd foldername` | Moves into a folder |
| `explorer .` | Opens current folder in Windows Explorer |

---

## The Everyday Workflow (Quick Reminder)

```
1. Edit your file
2. git add .
3. git commit -m "brief description of what you changed"
4. git push
```

---

## Connecting GitHub to Claude Projects

1. Edit your file locally
2. Run the everyday cycle above to save to GitHub
3. When Claude needs the updated context:
   - Go to your Claude Project
   - Delete the old version of the file from Project Knowledge
   - Upload the updated file
   - Claude will use the new version in future conversations

---

*Your repository: https://github.com/sheridaryan/agents-for-the-elderly*
