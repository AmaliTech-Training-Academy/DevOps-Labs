# **Git repository management lab**

**Learning objective:** Apply fundamental Git and GitHub commands to manage source code

**Bloom's level:** Apply

In this lab, you will create a Git repository, make changes, commit, branch, merge, and push your work to GitHub. By the end, you will have hands-on experience managing source code using the most essential Git commands.

---

### **Task 1: Configure Git (one-time setup)**

**Step 1 — Set your username**

```bash
git config --global user.name "Your Name"
```

**Step 2 — Set your email**

```bash
git config --global user.email "your@email.com"
```

**Step 3 — Confirm configuration**

```bash
git config --global --list
```

Expected output: You should see your name, email, and default settings.

---

### **Task 2: Initialize a local repository**

**Step 1 — Create a folder**

```bash
mkdir git-lab
cd git-lab
```

**Step 2 — Initialize Git**

```bash
git init
```

Expected output: Initialized empty Git repository in ...

---

### **Task 3: Create your first commit**

**Step 1 — Create a file**

```bash
echo "Hello Git!" > readme.txt
```

**Step 2 — Check file status**

```bash
git status
```

**Step 3 — Add file to staging**

```bash
git add readme.txt
```

**Step 4 — Commit**

```bash
git commit -m "Initial commit: Added readme file"
```

Expected result: A new commit is created and stored in the repository history.

---

### **Task 4: Create and work on a branch**

**Step 1 — Create a branch**

```bash
git branch feature-update
```

**Step 2 — Switch to it**

```bash
git checkout feature-update
```

**Step 3 — Edit your file**

```bash
echo "New feature added!" >> readme.txt
```

**Step 4 — Stage and commit**

```bash
git add readme.txt
git commit -m "Updated readme with new feature text"
```

Expected outcome: You now have work that exists only in your feature branch.

---

### **Task 5: Merge the branch back to main**

**Step 1 — Switch to main**

```bash
git checkout main
```

**Step 2 — Merge**

```bash
git merge feature-update
```

Expected result: Git merges the changes from feature-update into main.

---

### **Task 6: Connect to GitHub**

**Step 1 — Create a new GitHub repo**

Go to https://github.com/new and name it git-lab.

**Step 2 — Add GitHub as remote**

Copy your GitHub repo link and run:

```bash
git remote add origin https://github.com/<your-username>/git-lab.git
```

**Step 3 — Push your content**

```bash
git push -u origin main
```

Expected outcome: Your entire project appears in GitHub.

---

### **Task 7: View commit history**

**Step 1 — View history**

```bash
git log --oneline --graph
```

Expected output: A simple tree showing the main branch and your feature branch merge.

---

### **End-of-lab deliverables**

By the end of this lab, you should have:

- A local Git repository

- At least two commits

- A feature branch created and merged

- A GitHub repository with all changes pushed

- Understanding of staging, committing, branching, merging, and pushing

---

### **Optional challenge (for mastery)**

**Challenge 1: Undo a commit**

```bash
git revert <commit-hash>
```

**Challenge 2: Clone your GitHub repo somewhere else**

```bash
git clone https://github.com/<your-username>/git-lab.git
```

Work on it, push, and check your updates on GitHub.
