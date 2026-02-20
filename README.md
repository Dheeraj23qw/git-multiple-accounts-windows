<h1 align="center">One Machine, Two Identities: The Git & SSH Mastery Guide</h1>

<p align="center">
  <b>If you have a Work GitHub and a Personal GitHub on the same Windows laptop, you know the headache.</b><br>
  This guide explains how to make your computer <i>automatically</i> switch identities so you never commit work code with your personal email again.
</p>

---

<h2> The Struggle (The "Why")</h2>
<p>Most developers start with one GitHub account. But when you get a job, you get a second one. Suddenly:</p>
<ul>
  <li><b>Identity Crisis:</b> You commit code to your office repo, but it shows your personal email.</li>
  <li><b>Permission Denied:</b> GitHub gets confused about which SSH key belongs to which account.</li>
  <li><b>Manual Pain:</b> You find yourself typing <code>git config user.email "..."</code> every single time.</li>
</ul>

<p align="center">
  
</p>

---

<h2> Phase 1: Creating Your Digital IDs (SSH Keys)</h2>
<p>Think of SSH keys as your digital fingerprints. You need a unique one for each account.</p>

<pre><code># Create your Personal Key
ssh-keygen -t ed25519 -f ~/.ssh/id_self

# Create your Work Key
ssh-keygen -t ed25519 -f ~/.ssh/id_work
</code></pre>

<blockquote>
  <b>Note:</b> You will get two files for each: a <b>Private Key</b> (keep it secret!) and a <b>Public Key</b> (.pub). 
  Copy the content of the <code>.pub</code> files and paste them into your respective GitHub account settings.
</blockquote>

---

<h2> Phase 2: The "Key Manager" (SSH Agent)</h2>
<p>Your computer is forgetful. You must "hand" your keys to a manager (the SSH Agent) so it can show them to GitHub when you push code.</p>

<pre><code># 1. Start the Manager
eval $(ssh-agent -s)

# 2. Give the keys to the Manager
ssh-add ~/.ssh/id_self
ssh-add ~/.ssh/id_work
</code></pre>

<p align="center">
  
</p>

---

<h2> Phase 3: The "Translator" (SSH Config)</h2>
<p>GitHub's address is always <code>github.com</code>. We need to create "nicknames" so our computer knows which key to use.</p>

<p>Open or create <code>~/.ssh/config</code> and paste this:</p>

<pre><code># Personal Alias
Host github.com-self
    HostName github.com
    IdentityFile ~/.ssh/id_self

# Work Alias
Host github.com-work
    HostName github.com
    IdentityFile ~/.ssh/id_work
</code></pre>

---

<h2> Phase 4: The "Auto-Switcher" (Git Config)</h2>
<p>This is the magic part. We tell Git: <i>"If I am in my Work folder, use my Work email. Otherwise, use my Personal email."</i></p>

<p>In your main <code>~/.gitconfig</code> file:</p>

<pre><code>[user]
    name = Your Name
    email = personal@email.com

# THE MAGIC RULES
[includeIf "gitdir/i:C:/Users/rahul/work/"]
    path = ~/.gitconfig-work

[includeIf "gitdir/i:C:/Users/rahul/personal/"]
    path = ~/.gitconfig-personal
</code></pre>

<blockquote>
  <b>Crucial Point:</b> We use <code>gitdir/i:</code> because Windows is case-insensitive (it doesn't care about C: vs c:). The <code>/</code> at the end tells Git to include <i>everything</i> inside that folder.
</blockquote>

---

<h2>Phase 5: Testing Your Success</h2>
<p>Run these commands to see if the computer says "Hi" to the right person!</p>

<table>
  <tr>
    <th>Action</th>
    <th>Command</th>
    <th>Expected Result</th>
  </tr>
  <tr>
    <td>Test Personal</td>
    <td><code>ssh -T git@github.com-self</code></td>
    <td>"Hi PersonalUsername!"</td>
  </tr>
  <tr>
    <td>Test Work</td>
    <td><code>ssh -T git@github.com-work</code></td>
    <td>"Hi WorkUsername!"</td>
  </tr>
  <tr>
    <td>Verify Email</td>
    <td><code>git config user.email</code></td>
    <td>The correct email for that folder</td>
  </tr>
</table>

---

<h2> The Golden Rule for Daily Life</h2>
<p>When you clone a new project, <b>always use your nickname alias</b>. If you don't, Git won't know which key to use!</p>

<p><b>Bad way:</b> <code>git clone git@github.com:org/repo.git</code></p>
<p><b>Good way:</b> <code>git clone git@github.com-work:org/repo.git</code></p>

<hr>

<p align="center">
  <b>Built with ❤️ by a developer who survived the Git struggle.</b>
</p>

<h1 align="center"> The "All-Commands" Git & SSH Cheat Sheet</h1>

<p align="center">
  <b>A comprehensive list of every command required to set up and maintain a dual-identity Git environment.</b>
</p>

---

<h2> Phase 1: SSH Key Generation</h2>
<p>Run these to create your unique digital IDs for each account.</p>

<pre><code># Generate Personal Key
ssh-keygen -t ed25519 -f ~/.ssh/id_self

# Generate Work Key
ssh-keygen -t ed25519 -f ~/.ssh/id_work

# View Public Key (to copy into GitHub Settings)
cat ~/.ssh/id_self.pub
cat ~/.ssh/id_work.pub
</code></pre>

---

<h2> Phase 2: SSH Agent Management</h2>
<p>Commands to start the memory manager and load your keys for the day.</p>

<pre><code># Start the SSH Agent
eval $(ssh-agent -s)

# Load your specific keys into the Agent
ssh-add ~/.ssh/id_self
ssh-add ~/.ssh/id_work

# Verify which keys are currently loaded
ssh-add -l

# Delete all keys from memory (to reset)
ssh-add -D
</code></pre>

---

<h2> Phase 3: Connection Testing</h2>
<p>Verify that your <code>~/.ssh/config</code> aliases are working correctly.</p>

<pre><code># Test Personal Identity Handshake
ssh -T git@github.com-self

# Test Work Identity Handshake
ssh -T git@github.com-work
</code></pre>

---

<h2> Phase 4: Git Identity & Audit</h2>
<p>Commands to verify that your <code>includeIf</code> logic is swapping your name and email correctly.</p>

<pre><code># Check active email in current directory
git config user.email

# Audit exactly which file is providing the current settings
git config --list --show-origin

# See the 'Scope' of every setting (Global vs Local vs Work)
git config --list --show-scope

# Check current user name
git config user.name
</code></pre>

---

<h2> Phase 5: Repository Operations</h2>
<p>How to interact with projects using your new alias-based system.</p>

<pre><code># Clone a Work Repo using the alias
git clone git@github.com-work:OrgName/RepoName.git

# Clone a Personal Repo using the alias
git clone git@github.com-self:UserName/RepoName.git

# Update an existing project to use a specific alias
git remote set-url origin git@github.com-work:OrgName/RepoName.git

# Initialize a brand new folder to trigger includeIf logic
git init
</code></pre>

---

<h2> Master Command Reference Table</h2>

<table width="100%">
  <tr>
    <th align="left">Command</th>
    <th align="left">Usecase & Backend Function</th>
  </tr>
  <tr>
    <td><code>ssh-keygen</code></td>
    <td>Creates the encryption files used for identity.</td>
  </tr>
  <tr>
    <td><code>eval $(ssh-agent -s)</code></td>
    <td>Wakes up the background process that holds keys.</td>
  </tr>
  <tr>
    <td><code>ssh-add [path]</code></td>
    <td>Puts a key into "Active Duty" for the current session.</td>
  </tr>
  <tr>
    <td><code>ssh -T [alias]</code></td>
    <td>Asks GitHub: "Who do you think I am using this key?"</td>
  </tr>
  <tr>
    <td><code>git config --list --show-origin</code></td>
    <td>The "Debugger" command. Points to the file path of every active setting.</td>
  </tr>
  <tr>
    <td><code>git remote -v</code></td>
    <td>Shows if your project is using the <b>alias</b> or the <b>default</b> GitHub URL.</td>
  </tr>
  <tr>
    <td><code>ls -al ~/.ssh</code></td>
    <td>Lists all files in your SSH folder to verify keys exist.</td>
  </tr>
</table>

---

<p align="center">
  <b>Reference this sheet whenever you switch machines or set up a new environment.</b>
</p>**
