# Day 5: Git Version Control Part 1 📝

*"Tracking Every Change - The Foundation of Modern Development"*

**Estimated Time**: 3 hours  
**Difficulty**: Beginner to Intermediate  
**Prerequisites**: Day 4 completed, basic command line skills

---

## 🎯 Learning Objectives

By the end of today, you will:
- Understand version control concepts and why Git matters
- Master basic Git commands and workflows
- Create and manage Git repositories
- Understand branching and merging strategies
- Collaborate effectively using remote repositories
- Implement Git best practices for DevOps
- **Interview Ready**: Demonstrate Git proficiency in technical scenarios

---

## 📖 Theory Section: Why Version Control Changed Everything

### The Problem: Chaos Without Version Control

Imagine you're writing a book with 10 co-authors:
- **Without version control**: Everyone emails Word documents back and forth
  - "final_version.doc"
  - "final_version_v2.doc"
  - "final_version_v2_johns_edits.doc"
  - "final_version_v2_johns_edits_ACTUALLY_FINAL.doc"
  - **Result**: Chaos, lost work, conflicts

- **With version control**: Everyone works on the same project with complete history
  - Every change is tracked
  - You can see who changed what and when
  - You can revert to any previous version
  - Multiple people can work simultaneously without conflicts
  - **Result**: Organized, collaborative, safe

### What is Git?

Git is a **distributed version control system** created by Linus Torvalds (yes, the same person who created Linux) in 2005. Here's why it's revolutionary:

**Traditional Version Control** (like SVN):
- Central server stores everything
- If server goes down, nobody can work
- Slow operations (everything requires server communication)

**Git (Distributed)**:
- Every developer has complete history locally
- Work offline, sync when ready
- Fast operations (everything is local)
- Multiple backup copies naturally exist

### 💡 Did You Know?

**Git Powers the World**: Git manages the source code for:
- **Linux Kernel**: 25+ million lines of code, 20,000+ contributors
- **Facebook**: Billions of lines of code across thousands of repositories
- **Google**: Stores 2+ billion lines of code in a single repository
- **Your Career**: 87% of developers use Git (Stack Overflow Survey)

Companies pay premium salaries for Git expertise because it's the foundation of modern software delivery!

---

## 🏗️ Git Fundamentals: Understanding the Model

### The Three States of Git

Git has three main states that your files can be in:

```
Working Directory  →  Staging Area  →  Git Repository
     (Modified)         (Staged)        (Committed)
        │                   │               │
        │                   │               │
    Your files          Ready to          Permanently
    as you edit         commit            stored
```

**Working Directory**: Your actual files as you edit them
**Staging Area**: A holding area for changes you want to commit
**Repository**: The permanent record of your project's history

### Git Workflow: The Daily Cycle

```bash
# 1. Make changes to files
echo "Hello World" > hello.txt

# 2. Stage changes (prepare for commit)
git add hello.txt

# 3. Commit changes (save to repository)
git commit -m "Add hello world file"

# 4. Push to remote repository (share with team)
git push origin main
```

### Understanding Git Objects

Git stores everything as objects:

**Blob**: File content
**Tree**: Directory structure
**Commit**: Snapshot of your project at a point in time
**Tag**: Named reference to a specific commit

Think of commits like photographs of your project - each one captures the complete state at that moment.

---

## 🚀 Essential Git Commands

### Repository Setup

**Initialize a new repository**:
```bash
# Create new directory and initialize Git
mkdir my-project
cd my-project
git init

# Check status
git status
```

**Clone an existing repository**:
```bash
# Clone from GitHub
git clone https://github.com/username/repository.git

# Clone to specific directory
git clone https://github.com/username/repository.git my-local-name

# Clone specific branch
git clone -b develop https://github.com/username/repository.git
```

**Configure Git** (do this once per machine):
```bash
# Set your identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default editor
git config --global core.editor "vim"

# Set default branch name
git config --global init.defaultBranch main

# View configuration
git config --list
```

### Basic Git Workflow

**Check repository status**:
```bash
# See what's changed
git status

# Short status format
git status -s
```

**Stage changes**:
```bash
# Stage specific file
git add filename.txt

# Stage all changes
git add .

# Stage all files of specific type
git add *.js

# Interactive staging
git add -i

# Stage parts of a file
git add -p filename.txt
```

**Commit changes**:
```bash
# Commit with message
git commit -m "Add user authentication feature"

# Commit with detailed message
git commit -m "Add user authentication feature

- Implement login/logout functionality
- Add password hashing with bcrypt
- Create user session management
- Add input validation for forms"

# Commit all tracked changes (skip staging)
git commit -am "Quick fix for typo"

# Amend last commit
git commit --amend -m "Corrected commit message"
```

**View history**:
```bash
# View commit history
git log

# Compact one-line format
git log --oneline

# Show last 5 commits
git log -5

# Show commits with file changes
git log --stat

# Show commits with actual changes
git log -p

# Graphical representation
git log --graph --oneline --all
```

### Working with Remote Repositories

**Add remote repository**:
```bash
# Add origin remote
git remote add origin https://github.com/username/repository.git

# View remotes
git remote -v

# Add additional remote
git remote add upstream https://github.com/original/repository.git
```

**Push and pull changes**:
```bash
# Push to remote repository
git push origin main

# Push and set upstream
git push -u origin main

# Pull changes from remote
git pull origin main

# Fetch changes without merging
git fetch origin

# Push new branch
git push origin feature-branch
```

### Viewing Changes

**See what's changed**:
```bash
# Show unstaged changes
git diff

# Show staged changes
git diff --staged

# Show changes between commits
git diff HEAD~1 HEAD

# Show changes in specific file
git diff filename.txt

# Show changes between branches
git diff main feature-branch
```

**Show file content at specific commit**:
```bash
# Show file at specific commit
git show HEAD:filename.txt
git show abc123:filename.txt

# Show commit details
git show abc123

# Show files changed in commit
git show --name-only abc123
```

---

## 🌿 Branching: Git's Superpower

### Understanding Branches

Think of branches like parallel universes for your code:
- **main branch**: The stable, production-ready version
- **feature branches**: Experimental work that doesn't affect main
- **hotfix branches**: Quick fixes for urgent issues

```
main:     A---B---C---F---G
               \         /
feature:        D---E---/
```

### Basic Branching Commands

**Create and switch branches**:
```bash
# Create new branch
git branch feature-login

# Switch to branch
git checkout feature-login

# Create and switch in one command
git checkout -b feature-login

# Modern way (Git 2.23+)
git switch feature-login
git switch -c feature-login  # create and switch
```

**List and manage branches**:
```bash
# List local branches
git branch

# List all branches (including remote)
git branch -a

# List remote branches
git branch -r

# Delete branch
git branch -d feature-login

# Force delete branch
git branch -D feature-login

# Rename current branch
git branch -m new-branch-name
```

### Merging Branches

**Merge strategies**:
```bash
# Switch to target branch
git checkout main

# Merge feature branch
git merge feature-login

# Merge with no fast-forward (creates merge commit)
git merge --no-ff feature-login

# Merge with squash (combines all commits into one)
git merge --squash feature-login
```

**Understanding merge types**:

**Fast-forward merge** (when no changes on main):
```
Before:  main: A---B
         feature:   \---C---D

After:   main: A---B---C---D
```

**Three-way merge** (when both branches have changes):
```
Before:  main: A---B---E
         feature:   \---C---D

After:   main: A---B---E---M
                    \     /
         feature:     C---D
```

### Resolving Merge Conflicts

When Git can't automatically merge changes, you get a conflict:

```bash
# Attempt to merge
git merge feature-branch
# Output: CONFLICT (content): Merge conflict in file.txt

# View conflicted files
git status

# Edit conflicted file - you'll see:
<<<<<<< HEAD
This is the main branch version
=======
This is the feature branch version
>>>>>>> feature-branch

# Resolve by choosing one or combining both
# Remove conflict markers and save

# Stage resolved file
git add file.txt

# Complete the merge
git commit -m "Resolve merge conflict in file.txt"
```

### Real-World Branching Strategy: Git Flow

```bash
# Main branches
main        # Production-ready code
develop     # Integration branch

# Supporting branches
feature/*   # New features
release/*   # Prepare releases
hotfix/*    # Emergency fixes

# Example workflow
git checkout develop
git checkout -b feature/user-authentication
# ... work on feature ...
git checkout develop
git merge feature/user-authentication
git branch -d feature/user-authentication
```

---

## 🔄 Advanced Git Operations

### Undoing Changes

**Undo working directory changes**:
```bash
# Discard changes in specific file
git checkout -- filename.txt

# Discard all changes
git checkout -- .

# Modern way
git restore filename.txt
git restore .
```

**Undo staged changes**:
```bash
# Unstage specific file
git reset HEAD filename.txt

# Unstage all files
git reset HEAD

# Modern way
git restore --staged filename.txt
```

**Undo commits**:
```bash
# Undo last commit, keep changes
git reset --soft HEAD~1

# Undo last commit, unstage changes
git reset --mixed HEAD~1  # default

# Undo last commit, discard changes (DANGEROUS!)
git reset --hard HEAD~1

# Create new commit that undoes previous commit
git revert HEAD
```

### Stashing Changes

**Save work in progress**:
```bash
# Stash current changes
git stash

# Stash with message
git stash save "Work in progress on login feature"

# List stashes
git stash list

# Apply most recent stash
git stash apply

# Apply and remove stash
git stash pop

# Apply specific stash
git stash apply stash@{2}

# Drop stash
git stash drop stash@{1}

# Clear all stashes
git stash clear
```

### Rewriting History

**Interactive rebase** (rewrite commit history):
```bash
# Rebase last 3 commits
git rebase -i HEAD~3

# In the editor, you can:
# pick = use commit as-is
# reword = change commit message
# edit = modify commit
# squash = combine with previous commit
# drop = remove commit
```

**Cherry-pick** (apply specific commit to current branch):
```bash
# Apply commit from another branch
git cherry-pick abc123

# Apply multiple commits
git cherry-pick abc123 def456

# Apply commit without committing
git cherry-pick --no-commit abc123
```

---

## 🎯 Interview Preparation: Git Scenarios

### Question 1: Basic Git Workflow

**Interviewer**: "Walk me through the process of contributing to a project using Git."

**How to Think Through This**:
1. Start with cloning/forking
2. Create feature branch
3. Make changes and commit
4. Push and create pull request
5. Handle feedback and merge

**Sample Answer**:
```bash
# 1. Clone the repository
git clone https://github.com/company/project.git
cd project

# 2. Create feature branch
git checkout -b feature/add-user-validation

# 3. Make changes
# ... edit files ...

# 4. Stage and commit changes
git add .
git commit -m "Add email validation for user registration

- Implement regex validation for email format
- Add unit tests for validation function
- Update user registration form with validation"

# 5. Push branch to remote
git push -u origin feature/add-user-validation

# 6. Create pull request (via GitHub/GitLab UI)
# 7. Address review feedback
# 8. Merge when approved
```

**Follow-up Explanation**: "I follow a feature branch workflow where each new feature or bug fix gets its own branch. This keeps the main branch stable and allows for code review before merging. I write descriptive commit messages that explain both what changed and why."

### Question 2: Merge Conflict Resolution

**Interviewer**: "You're trying to merge a feature branch but encounter conflicts. How do you resolve them?"

**Sample Answer**:
```bash
# Attempt merge
git checkout main
git merge feature/user-auth

# Git reports conflicts
# CONFLICT (content): Merge conflict in src/auth.js

# Check which files have conflicts
git status

# Open conflicted file and look for conflict markers
# <<<<<<< HEAD
# main branch code
# =======
# feature branch code
# >>>>>>> feature/user-auth

# Resolve conflicts by:
# 1. Choosing one version
# 2. Combining both versions
# 3. Writing completely new code

# After resolving, stage the file
git add src/auth.js

# Complete the merge
git commit -m "Resolve merge conflict in auth.js

Combined authentication methods from both branches
to support both email and username login"

# Verify everything works
npm test
git push origin main
```

**Explanation**: "I resolve conflicts by understanding what each branch was trying to accomplish, then combining the changes in a way that preserves both intentions. I always test after resolving conflicts to ensure nothing broke."

### Question 3: Undoing Mistakes

**Interviewer**: "You accidentally committed sensitive information (like passwords) to Git. How do you fix this?"

**Sample Answer**:
```bash
# If it's the most recent commit and not pushed yet
git reset --soft HEAD~1  # Undo commit, keep changes
# Remove sensitive info from files
git add .
git commit -m "Add configuration without sensitive data"

# If it's already pushed or in older commits
# Use BFG Repo-Cleaner or git filter-branch
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch config/secrets.yml' \
  --prune-empty --tag-name-filter cat -- --all

# Force push to rewrite remote history
git push --force-with-lease origin main

# Alternative: Use BFG (easier)
java -jar bfg.jar --delete-files secrets.yml my-repo.git
cd my-repo.git
git reflog expire --expire=now --all && git gc --prune=now --aggressive
git push --force
```

**Important Note**: "After rewriting history, I'd immediately rotate any exposed credentials and notify the team about the force push. Prevention is better - I use .gitignore and environment variables for sensitive data."

### Question 4: Branch Strategy

**Interviewer**: "How would you set up a branching strategy for a team of 10 developers working on a web application?"

**Sample Answer**:
"I'd implement a modified Git Flow strategy:

**Main Branches**:
- `main`: Production-ready code, protected branch
- `develop`: Integration branch for features

**Supporting Branches**:
- `feature/*`: Individual features (e.g., `feature/user-dashboard`)
- `release/*`: Prepare releases (e.g., `release/v2.1.0`)
- `hotfix/*`: Emergency production fixes

**Workflow**:
```bash
# Feature development
git checkout develop
git checkout -b feature/payment-integration
# ... develop feature ...
git checkout develop
git merge --no-ff feature/payment-integration

# Release preparation
git checkout -b release/v2.1.0 develop
# ... bug fixes, version bumps ...
git checkout main
git merge --no-ff release/v2.1.0
git tag v2.1.0
git checkout develop
git merge --no-ff release/v2.1.0

# Hotfix
git checkout -b hotfix/security-patch main
# ... fix critical issue ...
git checkout main
git merge --no-ff hotfix/security-patch
git tag v2.1.1
git checkout develop
git merge --no-ff hotfix/security-patch
```

**Branch Protection Rules**:
- Require pull requests for main and develop
- Require status checks (CI/CD)
- Require code review from 2+ team members
- No direct pushes to protected branches"

### Question 5: Git Best Practices

**Interviewer**: "What Git best practices would you enforce on a development team?"

**Sample Answer**:
"Here are the key practices I'd implement:

**Commit Best Practices**:
```bash
# Good commit messages (follow conventional commits)
git commit -m "feat: add user authentication with JWT

- Implement login/logout endpoints
- Add JWT token generation and validation
- Create middleware for protected routes
- Add unit tests for auth functions

Closes #123"

# Atomic commits (one logical change per commit)
git add auth.js
git commit -m "feat: add JWT authentication"
git add tests/auth.test.js
git commit -m "test: add authentication unit tests"
```

**Repository Setup**:
```bash
# Comprehensive .gitignore
echo "node_modules/
.env
*.log
dist/
.DS_Store" > .gitignore

# Pre-commit hooks
npm install husky lint-staged --save-dev
# Configure to run linting and tests before commits
```

**Workflow Rules**:
1. **Never commit directly to main/develop**
2. **Use descriptive branch names**: `feature/user-auth`, not `fix-stuff`
3. **Squash commits** before merging to keep history clean
4. **Write meaningful commit messages** that explain why, not just what
5. **Review all code** through pull requests
6. **Keep commits small** and focused on single changes
7. **Use .gitignore** to exclude generated files and secrets
8. **Tag releases** with semantic versioning"

---

## 🎮 Hands-On Challenge: Git Collaboration Simulation

**Time**: 45 minutes  
**Scenario**: You're joining a development team and need to contribute to an existing project.

### Mission 1: Repository Setup (15 minutes)

```bash
# Create a sample project
mkdir git-collaboration-demo
cd git-collaboration-demo
git init

# Create initial files
echo "# My DevOps Project" > README.md
echo "node_modules/
.env
*.log" > .gitignore

mkdir -p src tests docs
echo "console.log('Hello DevOps!');" > src/app.js
echo "// Test file" > tests/app.test.js
echo "# Documentation" > docs/setup.md

# Initial commit
git add .
git commit -m "Initial project setup

- Add README and basic project structure
- Configure .gitignore for Node.js project
- Create src, tests, and docs directories"

# Create and push to remote (simulate GitHub)
# In real scenario: create repo on GitHub first
git branch -M main
# git remote add origin https://github.com/username/git-collaboration-demo.git
# git push -u origin main
```

### Mission 2: Feature Development (15 minutes)

```bash
# Create feature branch
git checkout -b feature/user-authentication

# Develop the feature
echo "class UserAuth {
  constructor() {
    this.users = [];
  }
  
  register(username, password) {
    // Hash password in real implementation
    this.users.push({ username, password });
    return true;
  }
  
  login(username, password) {
    const user = this.users.find(u => u.username === username);
    return user && user.password === password;
  }
}

module.exports = UserAuth;" > src/auth.js

# Add tests
echo "const UserAuth = require('../src/auth');

describe('UserAuth', () => {
  let auth;
  
  beforeEach(() => {
    auth = new UserAuth();
  });
  
  test('should register new user', () => {
    const result = auth.register('testuser', 'password123');
    expect(result).toBe(true);
  });
  
  test('should login with correct credentials', () => {
    auth.register('testuser', 'password123');
    const result = auth.login('testuser', 'password123');
    expect(result).toBe(true);
  });
  
  test('should fail login with incorrect credentials', () => {
    auth.register('testuser', 'password123');
    const result = auth.login('testuser', 'wrongpassword');
    expect(result).toBe(false);
  });
});" > tests/auth.test.js

# Update documentation
echo "# User Authentication

## Features
- User registration
- User login
- Password validation

## Usage
\`\`\`javascript
const UserAuth = require('./src/auth');
const auth = new UserAuth();

auth.register('username', 'password');
const isLoggedIn = auth.login('username', 'password');
\`\`\`" > docs/auth.md

# Commit changes
git add .
git commit -m "feat: implement user authentication system

- Add UserAuth class with register/login methods
- Include comprehensive unit tests
- Add documentation for authentication features

Closes #001"

# Push feature branch
# git push -u origin feature/user-authentication
```

### Mission 3: Collaboration Simulation (15 minutes)

```bash
# Simulate another developer's work on main
git checkout main

# Create conflicting changes (simulate teammate's work)
echo "# My DevOps Project

This project demonstrates DevOps best practices including:
- Version control with Git
- Automated testing
- Continuous integration
- Documentation

## Getting Started
1. Clone the repository
2. Install dependencies
3. Run tests" > README.md

git add README.md
git commit -m "docs: enhance README with project description"

# Try to merge feature branch (will create conflict)
git merge feature/user-authentication

# Resolve conflict in README.md
echo "# My DevOps Project

This project demonstrates DevOps best practices including:
- Version control with Git
- User authentication system
- Automated testing
- Continuous integration
- Documentation

## Features
- User registration and login
- Password validation
- Comprehensive test suite

## Getting Started
1. Clone the repository
2. Install dependencies
3. Run tests" > README.md

# Complete the merge
git add README.md
git commit -m "Merge feature/user-authentication into main

Resolved conflict in README.md by combining project description
with new authentication features documentation"

# Clean up feature branch
git branch -d feature/user-authentication

# View the history
git log --oneline --graph

# Create a release tag
git tag -a v1.0.0 -m "Release version 1.0.0

Features:
- User authentication system
- Comprehensive test suite
- Project documentation"

git tag
```

---

## 📚 Additional Resources

### Essential Git Resources

**Books**:
- **"Pro Git" by Scott Chacon**: Free online, comprehensive Git guide
- **"Git Pocket Guide" by Richard Silverman**: Quick reference for daily use
- **"Version Control with Git" by Jon Loeliger**: Deep dive into Git internals

**Online Resources**:
- **Git Documentation**: Official Git reference and tutorials
- **GitHub Learning Lab**: Interactive Git tutorials
- **Atlassian Git Tutorials**: Comprehensive guides with visual examples

**Practice Platforms**:
- **Learn Git Branching**: Interactive visual Git tutorial
- **Git Immersion**: Hands-on Git workshop
- **Katacoda Git Scenarios**: Browser-based Git practice

### Git Configuration Best Practices

**Global Git Configuration**:
```bash
# Essential settings
git config --global user.name "Your Name"
git config --global user.email "your.email@company.com"
git config --global init.defaultBranch main
git config --global core.editor "vim"

# Helpful aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'

# Better diff and merge tools
git config --global merge.tool vimdiff
git config --global diff.tool vimdiff

# Color output
git config --global color.ui auto
```

**Project-specific .gitignore templates**:
```bash
# Node.js
node_modules/
npm-debug.log*
.env

# Python
__pycache__/
*.pyc
.venv/

# Java
*.class
target/

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store
Thumbs.db
```

---

## 🔄 Day 5 Assessment

### Git Fundamentals Check (10 minutes)

**Challenge 1**: Explain the difference between `git pull` and `git fetch`.

<details>
<summary>Click for answer</summary>

**git fetch**: Downloads changes from remote repository but doesn't merge them into your current branch. You can review changes before merging.

**git pull**: Downloads changes AND automatically merges them into your current branch. Equivalent to `git fetch` followed by `git merge`.

Best practice: Use `git fetch` first to review changes, then `git merge` when ready.
</details>

**Challenge 2**: You have uncommitted changes and need to switch branches urgently. What are your options?

<details>
<summary>Click for answer</summary>

1. **Commit the changes**: `git add . && git commit -m "WIP: work in progress"`
2. **Stash the changes**: `git stash save "work in progress"`
3. **Discard the changes**: `git checkout -- .` (if you don't need them)

After switching branches and completing urgent work, you can:
- Apply stash: `git stash pop`
- Continue from WIP commit: `git reset --soft HEAD~1`
</details>

### Practical Git Scenario (15 minutes)

**Scenario**: You're working on a feature branch and realize you need to incorporate changes from the main branch. The main branch has moved forward since you started your feature.

**Your options and when to use each**:

<details>
<summary>Sample Solution</summary>

**Option 1: Merge main into feature branch**
```bash
git checkout feature-branch
git merge main
# Creates merge commit, preserves history
```

**Option 2: Rebase feature branch onto main**
```bash
git checkout feature-branch
git rebase main
# Replays your commits on top of main, cleaner history
```

**Option 3: Interactive rebase (if you want to clean up commits)**
```bash
git checkout feature-branch
git rebase -i main
# Allows squashing, reordering, editing commits
```

**When to use each**:
- **Merge**: When you want to preserve exact history and context
- **Rebase**: When you want clean, linear history (preferred for feature branches)
- **Interactive rebase**: When you want to clean up commit history before merging
</details>

---

## 🚀 Tomorrow's Preview: Git Version Control Part 2

Tomorrow we'll dive deeper into advanced Git concepts:

- **Advanced branching strategies** (Git Flow, GitHub Flow, GitLab Flow)
- **Git hooks and automation** (pre-commit, post-receive hooks)
- **Collaborative workflows** (pull requests, code review, conflict resolution)
- **Git in CI/CD pipelines** (automated testing, deployment triggers)
- **Git security and best practices** (signing commits, protecting sensitive data)
- **Troubleshooting Git issues** (recovering lost commits, fixing repository problems)

**Preparation for Tomorrow**:
- Practice today's Git commands until they're muscle memory
- Set up a GitHub account if you don't have one
- Try collaborating on a small project with Git
- Think about how Git fits into the DevOps workflow

---

## 🎉 Congratulations!

You've completed Day 5 and mastered Git fundamentals! You now understand:
- ✅ Version control concepts and Git's distributed model
- ✅ Essential Git commands for daily development work
- ✅ Branching and merging strategies
- ✅ Collaboration workflows with remote repositories
- ✅ Conflict resolution and history management
- ✅ Git best practices for professional development

**Key Takeaway**: Git is the foundation of modern software development and DevOps. Every code change, configuration update, and infrastructure modification should be version controlled. Master Git, and you master the ability to collaborate safely and track every change in your systems.

**Tomorrow**: We'll explore advanced Git workflows and see how Git integrates with CI/CD pipelines!

---

*"Git doesn't just track your code; it tracks your thinking. Every commit tells the story of how your solution evolved."* - Ready to master advanced Git workflows? 🚀

### **[🎯 Continue to Day 6: Git Version Control Part 2 →](day06.md)**