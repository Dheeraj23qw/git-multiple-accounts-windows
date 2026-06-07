````markdown
# One Machine, Two GitHub Identities

### The Complete Git + SSH Setup Guide for Work & Personal Accounts

---

# Why This Guide Exists

Most developers start with a single GitHub account.

Then one day:

- You join a company.
- You receive a second GitHub account.
- Everything starts breaking.

Common problems:

- Work commits show your personal email
- GitHub rejects SSH authentication
- Wrong account appears in pull requests
- Constantly changing `git config user.email`
- Accidental commits under the wrong identity

This guide solves all of those problems permanently.

---

# Architecture Overview

We'll create:

```text
┌────────────────────┐
│ Personal GitHub    │
│ SSH Key: id_self   │
└──────────┬─────────┘
           │
           ▼

       github.com-self

           ▲
           │

┌──────────┴─────────┐
│ Work GitHub        │
│ SSH Key: id_work   │
└────────────────────┘

            │
            ▼

     SSH Config Aliases

            │
            ▼

      Automatic Git
       Identity Switch
````

Result:

| Folder                        | Git Identity     |
| ----------------------------- | ---------------- |
| `C:/Users/Dheeraj/personal/*` | Personal Account |
| `C:/Users/Dheeraj/work/*`     | Work Account     |

No manual switching required.

---

# Phase 1 — Create SSH Keys

Generate separate SSH identities.

## Personal

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_self
```

## Work

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_work
```

You will get:

```text
id_self
id_self.pub

id_work
id_work.pub
```

### Public vs Private Key

| File         | Purpose          |
| ------------ | ---------------- |
| `.pub`       | Upload to GitHub |
| No extension | Keep secret      |

Upload:

GitHub → Settings → SSH and GPG Keys → New SSH Key

Add:

* `id_self.pub` → Personal GitHub
* `id_work.pub` → Work GitHub

---

# Phase 2 — Start SSH Agent

The SSH Agent remembers your keys.

```bash
eval $(ssh-agent -s)
```

Load both identities:

```bash
ssh-add ~/.ssh/id_self

ssh-add ~/.ssh/id_work
```

Verify:

```bash
ssh-add -l
```

Expected:

```text
2 identities loaded
```

---

# Phase 3 — Configure SSH Aliases

Create:

```text
~/.ssh/config
```

Add:

```config
# Personal Account

Host github.com-self
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_self
    IdentitiesOnly yes

# Work Account

Host github.com-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_work
    IdentitiesOnly yes
```

These aliases act like shortcuts.

| Alias           | Account  |
| --------------- | -------- |
| github.com-self | Personal |
| github.com-work | Work     |

---

# Phase 4 — Automatic Git Identity Switching

This is where the magic happens.

Main config:

```ini
[user]
    name = Dheeraj
    email = dheeraj9508820247@gmail.com

[includeIf "gitdir/i:C:/Users/Dheeraj/work/"]
    path = ~/.gitconfig-work

[includeIf "gitdir/i:C:/Users/Dheeraj/personal/"]
    path = ~/.gitconfig-personal
```

---

## Create Work Profile

File:

```text
~/.gitconfig-work
```

Content:

```ini
[user]
    email = codewithdheeraj19@gmail.com
```

---

## Create Personal Profile

File:

```text
~/.gitconfig-personal
```

Content:

```ini
[user]
    email = dheeraj9508820247@gmail.com
```

---

# Phase 5 — Test Everything

## Test Personal Account

```bash
ssh -T git@github.com-self
```

Expected:

```text
Hi YourPersonalUsername!
```

---

## Test Work Account

```bash
ssh -T git@github.com-work
```

Expected:

```text
Hi YourWorkUsername!
```

---

## Verify Active Email

```bash
git config user.email
```

Should match the folder you're currently inside.

---

# Golden Rule

Always clone using aliases.

### Wrong

```bash
git clone git@github.com:Org/repo.git
```

### Correct

```bash
git clone git@github.com-work:Org/repo.git
```

or

```bash
git clone git@github.com-self:username/repo.git
```

---

# Debugging Commands

## See Current Email

```bash
git config user.email
```

## See Current Name

```bash
git config user.name
```

## See Where Settings Come From

```bash
git config --list --show-origin
```

## Check Loaded SSH Keys

```bash
ssh-add -l
```

## Check Remote URL

```bash
git remote -v
```

## View SSH Directory

```bash
ls -al ~/.ssh
```

---

# Common Mistakes

### SSH Works But Git Uses Wrong Email

Cause:

```text
Repository not inside configured folder
```

Fix:

```text
Move repo into:

C:/Users/Dheeraj/work/

or

C:/Users/Dheeraj/personal/
```

---

### Permission Denied (publickey)

Cause:

```text
SSH key not added to agent
```

Fix:

```bash
ssh-add ~/.ssh/id_self
ssh-add ~/.ssh/id_work
```

---

### Wrong GitHub Account Appears

Cause:

```text
Remote URL uses github.com
instead of alias.
```

Fix:

```bash
git remote set-url origin git@github.com-work:Org/repo.git
```

---

# Final Folder Structure

```text
C:/Users/Dheeraj/

├── personal/
│   ├── project-a
│   ├── project-b
│   └── project-c
│
├── work/
│   ├── company-app
│   ├── internal-api
│   └── dashboard
│
└── .ssh/
    ├── config
    ├── id_self
    ├── id_self.pub
    ├── id_work
    └── id_work.pub
```

---

# Success

You now have:

* Separate SSH keys
* Separate GitHub accounts
* Automatic email switching
* Zero manual configuration per project
* Professional multi-account workflow

---

```
```
