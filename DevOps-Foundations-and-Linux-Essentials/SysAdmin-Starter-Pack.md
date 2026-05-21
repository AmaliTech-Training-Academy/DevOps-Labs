# **The SysAdmin starter pack**

**Goal:** Create and run a Bash script that automates common Linux administrative tasks, including file management, directory organization, and permission settings.

---

## **Learning objectives**

- Execute common Linux commands to navigate and manage the filesystem

- Create and manage files and directories using Bash commands

- Modify file permissions, ownership, and access using chmod, chown, and ls -l

- Automate routine system tasks with a Bash script

---

## **Challenge scenario**

You've just joined a DevOps team and been asked to set up a basic folder structure for storing project logs, configuration files, and scripts. To save time in the future, you'll automate this setup using a Bash script.

---

## **Tasks**

### **1. Setup — create a working directory**

```bash
mkdir ~/devops_challenge && cd ~/devops_challenge
```

### **2. Script creation — create a new script file**

```bash
nano setup_environment.sh
```

Inside the script, include commands that do the following:

- Create directories: logs/, configs/, scripts/

- Create files: logs/system.log, configs/app.conf, scripts/backup.sh

- Add sample content using echo commands

- Modify permissions using chmod as per the requirements below

- Display the directory structure and permissions using tree and ls -lR

---

## **Permissions requirements**

| **File** | **Permission** | **chmod value** |
| --- | --- | --- |
| logs/system.log | Owner read/write, group read, others read | 644 |
| configs/app.conf | Read-only for all | 444 |
| scripts/backup.sh | Owner read/write/execute, group read/execute, others read/execute | 755 |

---

## **Challenge extension**

Add logic to your script so that if the directories already exist, it does not recreate them but instead prints:

```
Directory already exists: [directory_name]
```

---

## **Expected output**

**Directory structure:**

```
.
├── configs
│   └── app.conf
├── logs
│   └── system.log
└── scripts
    └── backup.sh
```

**Permissions overview:**

```
-rw-r--r-- 1 user user 22 Oct 27 21:15 logs/system.log
-r--r--r-- 1 user user 24 Oct 27 21:15 configs/app.conf
-rwxr-xr-x 1 user user 48 Oct 27 21:15 scripts/backup.sh
```

---

## **Assessment criteria**

| **Criteria** | **Description** | **Points** |
| --- | --- | --- |
| Script executes successfully | No syntax errors and completes all tasks | 20 |
| Correct directory and file structure | All expected paths exist | 20 |
| Permissions set correctly | Matches requirements | 20 |
| Script handles existing files gracefully | Uses conditional logic | 20 |
| Script output clear and professional | Organized and readable | 20 |
| **Total** |  | **100** |

---

## **Submission**

Submit a GitHub link to a repository containing:

- Your script files

- Documented screenshots of the execution of your scripts

All screenshots must be of your entire screen with the date and time visible.
