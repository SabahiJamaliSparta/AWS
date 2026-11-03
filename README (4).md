# Deploy MongoDB on an Ubuntu EC2 Instance

## Overview

This guide records the installation of MongoDB Community Edition on an AWS EC2 instance, accessed through SSH from a Mac. MongoDB stores data as documents; `mongod` runs the database server, while `mongosh` is the shell used to interact with it [1, 3].

**Verified environment:** Ubuntu 26.04.1 LTS (`resolute`), x86_64; MongoDB **9.0.4**; MongoDB Shell **2.14.1**. MongoDB lists Ubuntu 26.04 support from version 9.0.4 [2].

## Before starting

Use a running Ubuntu EC2 instance with internet access, SSH access and an account with `sudo` privileges. Connect from the Mac using your own key and public DNS:

```bash
ssh -i ~/.ssh/YOUR_KEY.pem ubuntu@YOUR_PUBLIC_DNS
```

Replace both placeholders. **Run all installation commands below in the connected Ubuntu terminal**, rather than the Mac's local shell. This exercise installs directly on Ubuntu using APT.

## 1. Refresh Ubuntu's package lists

```bash
sudo apt-get update
```

This refreshes information about available packages without upgrading installed software. The completed run ended with `Reading package lists... Done`.

![Ubuntu retrieves package information](images/01-package-update.png)

## 2. Prepare the download and verification tools

```bash
sudo apt-get install -y gnupg curl
```

`curl` downloads the signing key; `gnupg` converts it into a usable keyring. `-y` confirms installation automatically. Both tools were already installed, so no new packages were needed.

## 3. Import MongoDB's signing key

```bash
curl -fsSL https://pgp.mongodb.com/server-9.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-9.gpg --dearmor
```

The public signing key lets APT verify packages from MongoDB. The command normally completes silently. Check that the key file exists:

```bash
ls -l /usr/share/keyrings/mongodb-server-9.gpg
```

**Recorded result:** A non-empty file of **2,417 bytes**. File presence confirms it was saved; the later repository refresh confirms APT accepted its signature.

![Preparation tools already installed and signing key saved](images/02-tools-and-key.png)

## 4. Add the MongoDB repository

```bash
echo "deb [ arch=amd64 signed-by=/usr/share/keyrings/mongodb-server-9.gpg ] https://repo.mongodb.org/apt/ubuntu resolute/mongodb-org/9.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-9.0.list
```

This adds the MongoDB 9.0 package source for Ubuntu 26.04 (`resolute`) on x86_64 (`amd64`). `signed-by` links the repository to the saved key. The terminal prints the repository entry after saving it.

Refresh the package lists again so APT includes the new source:

```bash
sudo apt-get update
```

**Recorded result:** MongoDB repository metadata and package lists downloaded without errors.

![APT successfully reads the MongoDB repository](images/03-repository-update.png)

## 5. Install MongoDB

```bash
sudo apt-get install -y mongodb-org
```

The `mongodb-org` package installs the server, shell and supporting tools. The repository supplied MongoDB **9.0.4** during this exercise; future installations may receive a newer 9.0 patch release.

![MongoDB server, shell and tools finish installing](images/04-installation-complete.png)

## 6. Start the database server

```bash
sudo systemctl start mongod
```

This starts the `mongod` service. No output is normally expected. Check its status:

```bash
sudo systemctl status mongod --no-pager
```

`--no-pager` prints the status directly in the terminal.

**Recorded result:** `Active: active (running)` confirms the server is running. The separate `disabled` label refers to automatic startup at boot, which was not enabled for this temporary exercise.

![MongoDB service reports active and running](images/05-service-running.png)

## 7. Connect with MongoDB Shell

```bash
mongosh
```

Run this from the Ubuntu shell. It opens a database connection on the EC2 instance [3].

**Recorded results:**

| Output | Meaning |
|---|---|
| `mongodb://127.0.0.1:27017/...` | Connection to MongoDB on the same EC2 instance. |
| `Using MongoDB: 9.0.4` | Server version reached successfully. |
| `Using Mongosh: 2.14.1` | Shell version in use. |
| `test>` | Shell is connected and ready for database commands. |

![MongoDB Shell connects and reports the installed versions](images/06-shell-connected.png)

![The connected MongoDB Shell prompt](images/07-shell-prompt.png)

**Deployment verified:** The server is running and the shell connects successfully. No database creation or data insertion was performed.

## Startup warnings and scope

The shell displayed filesystem and memory-configuration recommendations, plus a warning that access control was not enabled. These did not prevent the connection. This was a temporary learning deployment using local database access, not a production configuration. Keep database port **27017** closed to public access; authentication and other security controls are needed before wider use [4].

The exercise ends at the successful shell connection. Automatic startup was skipped because the instance is intended for deletion afterwards. Instance deletion is not evidenced here.

## References

1. MongoDB (n.d.) [Install MongoDB Community on Linux — Ubuntu, Package Manager](https://www.mongodb.com/docs/manual/administration/install-community-linux/?linux-distro=ubuntu&linux-method=pkg).
2. MongoDB (n.d.) [Supported Platforms for MongoDB Community](https://www.mongodb.com/docs/community-platform-support/).
3. MongoDB (n.d.) [Welcome to MongoDB Shell (`mongosh`)](https://www.mongodb.com/docs/mongodb-shell/).
4. MongoDB (n.d.) [Security](https://www.mongodb.com/docs/manual/security/).

All references accessed 3 November 2026. Screenshots and recorded results are from the completed exercise.
