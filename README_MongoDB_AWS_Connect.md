# Install MongoDB on AWS EC2

Install MongoDB on an Ubuntu EC2 instance and verify the connection. We used a Mac terminal to access the instance. Run the commands below **on the connected Ubuntu instance**.

**Used in this exercise:** Ubuntu 26.04.1, MongoDB 9.0.4 and MongoDB Shell 2.14.1. These steps follow MongoDB's Ubuntu installation guidance [1, 2].

## Before starting

You need an Ubuntu 26.04 EC2 instance with internet access and an SSH connection from your Mac. Your Ubuntu account must be allowed to use `sudo` to install software and start MongoDB.

## 1. Update the software list

```bash
sudo apt-get update
```

This lets Ubuntu check which software packages are available. It does not upgrade installed software.

**Result:** The command finished with `Reading package lists... Done`.

```text
Fetched 32.7 MB in 4s (7864 kB/s)
Reading package lists... Done
```

## 2. Check the tools we need

```bash
sudo apt-get install -y gnupg curl
```

These tools download and prepare MongoDB's verification key. Both were already installed on our instance.

**Output excerpt:**

```text
gnupg is already the newest version (2.4.8-4ubuntu3.1).
curl is already the newest version (8.18.0-1ubuntu2.8).
0 upgraded, 0 newly installed, 0 to remove and 31 not upgraded.
```

## 3. Add MongoDB's verification key

```bash
curl -fsSL https://pgp.mongodb.com/server-9.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-9.gpg --dearmor
```

This saves the public key used to check that MongoDB packages are genuine. The command normally finishes without displaying a message.

**Verification check:** Confirm the key file was saved:

```bash
ls -l /usr/share/keyrings/mongodb-server-9.gpg
```

**Result:** The key file existed and was **2,417 bytes**.

```text
-rw-r--r-- 1 root root 2417 Nov 3 15:20 /usr/share/keyrings/mongodb-server-9.gpg
```

## 4. Tell Ubuntu where to find MongoDB

Copy this as one command:

```bash
echo "deb [ arch=amd64 signed-by=/usr/share/keyrings/mongodb-server-9.gpg ] https://repo.mongodb.org/apt/ubuntu resolute/mongodb-org/9.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-9.0.list
```

This adds MongoDB's software source for our Ubuntu version. Refresh the software list again to include it:

```bash
sudo apt-get update
```

**Result:** Ubuntu downloaded the MongoDB package list without errors.

**Output excerpt:**

```text
Get:5 https://repo.mongodb.org/apt/ubuntu resolute/mongodb-org/9.0 InRelease [3008 B]
Get:6 https://repo.mongodb.org/apt/ubuntu resolute/mongodb-org/9.0/multiverse amd64 Packages [6830 B]
Fetched 9838 B in 1s (19.2 kB/s)
Reading package lists... Done
```

## 5. Install MongoDB

```bash
sudo apt-get install -y mongodb-org
```

This installs MongoDB and the tools needed to use it.

**Result:** MongoDB **9.0.4** and MongoDB Shell **2.14.1** were installed. A later run may install a newer 9.0 patch version.

**Output excerpt:**

```text
Setting up mongodb-mongosh (2.14.1) ...
Setting up mongodb-org-server (9.0.4) ...
Setting up mongodb-org (9.0.4) ...
```

## 6. Start MongoDB and check it is running

```bash
sudo systemctl start mongod
```

The start command normally finishes silently. Then check the status:

```bash
sudo systemctl status mongod --no-pager
```

**Result:** `active (running)` confirmed MongoDB was running.

**Output excerpt:**

```text
● mongod.service - MongoDB Database Server
     Loaded: loaded (/usr/lib/systemd/system/mongod.service; disabled; preset: enabled)
     Active: active (running) since Tue 2026-11-03 15:28:59 UTC; 4s ago
```

## 7. Connect to MongoDB

```bash
mongosh
```

This opens MongoDB's command-line shell [3].

**Result:** It connected to `127.0.0.1:27017`, meaning MongoDB on the same EC2 instance. The output showed the installed versions, followed by the `test>` prompt.

**Output excerpt:**

```text
Connecting to: mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.14.1
Using MongoDB: 9.0.4
Using Mongosh: 2.14.1
```

After the startup messages, the shell displayed:

```text
test>
```

**Task complete:** MongoDB was installed, started and successfully connected. No database or sample data was added. Automatic startup was skipped because this was a temporary instance.

## Terms used in the commands

| Term | Simple meaning |
|---|---|
| EC2 instance | A remote computer provided by AWS. |
| `sudo` | Runs a command with administrator permissions. We used it to install software, save system files and start MongoDB. |
| `apt-get` | Ubuntu's tool for finding and installing software. |
| `-y` | Automatically confirms installation. |
| `curl` | Downloads content from a web address. |
| `gnupg` / `gpg` | Tools used here to prepare the public verification key. |
| Repository | A source from which Ubuntu downloads software. |
| `resolute` | The name used for Ubuntu 26.04 in package sources. |
| `amd64` | The x86_64 processor architecture used by our instance. |
| `signed-by` | Specifies the key Ubuntu should use to verify this source. |
| `tee` | Writes the repository entry to a file and displays it. |
| `mongod` | The MongoDB database server. |
| `systemctl` | Starts services and checks their status. |
| `--no-pager` | Displays the status directly in the terminal. |
| `mongosh` | The command-line tool for interacting with MongoDB. |

## Notes on the output

The `disabled` service label meant automatic startup was off; it did not mean MongoDB had failed. The `active (running)` line confirmed its current state.

The shell also displayed configuration warnings, including that access control was off. We connected locally for this exercise. Keep database port **27017** closed to public access; additional security configuration is required before wider use [4].

## References

1. MongoDB (n.d.) [Install MongoDB Community on Linux: Ubuntu, Package Manager](https://www.mongodb.com/docs/manual/administration/install-community-linux/?linux-distro=ubuntu&linux-method=pkg).
2. MongoDB (n.d.) [Supported Platforms for MongoDB Community](https://www.mongodb.com/docs/community-platform-support/). Ubuntu 26.04 is supported from MongoDB 9.0.4.
3. MongoDB (n.d.) [MongoDB Shell](https://www.mongodb.com/docs/mongodb-shell/).
4. MongoDB (n.d.) [Security](https://www.mongodb.com/docs/manual/security/).

Accessed 3 November 2026. Output excerpts are from this exercise; spacing has been standardised for readability.