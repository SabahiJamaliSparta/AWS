# Flask on AWS EC2 — Mac setup guide

This task creates a simple Python web page that displays **Hello World!**. We wrote the code in PyCharm on a Mac, copied it to an Ubuntu computer on AWS, and opened the page from the Mac’s browser.

The app runs on the EC2 instance. The Mac is used to write the code, send commands and view the result. MongoDB is not needed for this Flask app.

## What we used

| Tool | Why we used it |
| --- | --- |
| Python and Flask | Create the web page and respond to browser requests. |
| PyCharm | Write and test `app.py` on the Mac. |
| AWS EC2 with Ubuntu | Run the app in the cloud. Our instance used Ubuntu 26.04.1 LTS and `t3.micro` in Frankfurt. |
| Mac Terminal, SCP and SSH | Copy the file to EC2 and run commands there. |
| Python virtual environment | Keep the app’s Python packages separate from Ubuntu’s system packages. |
| Browser | Check that the cloud app displays the expected page. |

## Before starting

Have a running Ubuntu EC2 instance with its status checks passed, and the private key selected when it was created. Our key file was `DataStudents.pem`, stored in `~/.ssh/` on the Mac.

In the commands below, replace `YOUR_PUBLIC_IP` with the **current Public IPv4 address** from the EC2 instance summary. Replace `YOUR_MAC_USERNAME` with your Mac username and adjust the project path if necessary. Do not type terminal prompts such as `$` or `%`.

Keep the `.pem` private key out of GitHub.

## 1. Write the Flask app — on the Mac

Create `app.py` in your PyCharm project:

```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello_world():
    return 'Hello World!'


if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000)
```

The code follows Flask’s minimal application pattern [1].

| Code | What it does |
| --- | --- |
| `from flask import Flask` | Imports the tool used to create the app. |
| `Flask(__name__)` | Creates the app and tells Flask which Python module it belongs to. |
| `@app.route('/')` | Connects the home page address `/` to the function below it. |
| `return 'Hello World!'` | Sends this text back to the browser. |
| `if __name__ == '__main__':` | Starts the server when this file is run directly. |
| `host="0.0.0.0"` | Makes the server listen on all of the instance’s network interfaces so outside requests can reach it. |
| `port=5000` | Uses port 5000 for this app. |

We first tested the app locally in PyCharm. That confirmed the code worked on the Mac; the next steps deploy it to AWS. Use the EC2 public address when testing the cloud version, rather than the Mac’s local address.

## 2. Check the EC2 security group — in AWS

Open **EC2 → Instances → your instance → Security**, then open the attached security group. We reused our existing group.

Select **Edit inbound rules** and ensure these rules exist [3]:

| Type | Protocol | Port | Source | Purpose |
| --- | --- | --- | --- | --- |
| SSH | TCP | 22 | My IP | Terminal access and file upload. |
| Custom TCP | TCP | 5000 | My IP | Browser access to Flask. |

Save the rules. **My IP** allows access from the current internet connection. If you change networks, update it. This setup was tested from home; it does not allow every internet user to access the page. Port 27017 belonged to the earlier MongoDB task and is not required here.

## 3. Upload `app.py` — in Mac Terminal

Run this from your Mac, where the file is stored:

```bash
scp -o ConnectTimeout=10 -i ~/.ssh/DataStudents.pem /Users/YOUR_MAC_USERNAME/PycharmProjects/WelcomeScreen/app.py ubuntu@YOUR_PUBLIC_IP:~/
```

`-i` selects the key file. `ConnectTimeout=10` limits the initial connection wait to ten seconds. The final `:~/` places the file in the Ubuntu user’s home folder.

Our successful upload showed:

```text
app.py    100%    179
```

You only need to upload again if you change the local code.

## 4. Connect to EC2 — in Mac Terminal

```bash
ssh -o ConnectTimeout=10 -i ~/.ssh/DataStudents.pem ubuntu@YOUR_PUBLIC_IP
```

SSH opens a terminal session on Ubuntu [4]. On a first connection, verify the host fingerprint before accepting the prompt. A successful connection displays the Ubuntu welcome message and an `ubuntu@...:~$` prompt.

**Run the remaining terminal commands on Ubuntu inside this SSH session.**

Check that the upload arrived:

```bash
ls -l ~/app.py
```

Our output confirmed a 179-byte file at `/home/ubuntu/app.py`.

## 5. Prepare Python — on Ubuntu

```bash
sudo apt-get update
sudo apt-get install -y python3-venv
```

The first command refreshes Ubuntu’s list of available software. The second installs the tool for creating Python virtual environments. `-y` automatically accepts the installation prompt.

Create and activate the environment [2]:

```bash
python3 -m venv venv
source venv/bin/activate
```

The first command creates a folder called `venv`. The second makes this terminal use its Python tools. Your prompt should now begin with `(venv)`. These commands usually produce no other output.

## 6. Install Flask — on Ubuntu

With `(venv)` visible, run:

```bash
python -m pip install flask
```

This installs Flask and its required packages into the virtual environment [2]. Our output confirmed **Flask 3.1.3** was installed successfully. A later installation may use a different version.

## 7. Start the app — on Ubuntu

```bash
python app.py
```

This runs the uploaded code. Check that Flask reports it is listening on port 5000. Leave the terminal open while testing: the app runs in this terminal session.

Flask’s development-server warning is expected. This exercise uses the built-in server with debug mode off for a temporary learning demonstration; a production deployment needs a production server [1]. We did not configure automatic startup or a background service.

## 8. Open the cloud app — in the Mac browser

Enter the full address, including **http** and **:5000**:

```text
http://YOUR_PUBLIC_IP:5000
```

Expected page:

```text
Hello World!
```

Our browser displayed this message successfully. This confirmed the file upload, Python setup, running server and security-group rule worked together. The browser sends a request to EC2 on port 5000; Flask matches `/`, runs `hello_world()` and returns the text.

The browser may label the page **Not Secure** because this demonstration uses HTTP without HTTPS encryption.

## Running the app again

If the app has stopped, reconnect from the Mac:

```bash
ssh -o ConnectTimeout=10 -i ~/.ssh/DataStudents.pem ubuntu@YOUR_PUBLIC_IP
```

Then, on Ubuntu:

```bash
cd ~
source venv/bin/activate
python app.py
```

You do not need to recreate the environment or reinstall Flask on every connection. Press **Control + C** to stop the running app. If the instance was recreated, repeat the upload and installation steps on the new instance.

## Troubleshooting

| Problem | What to check or do |
| --- | --- |
| SSH or SCP times out | Copy the current public IP from the correct running instance. Our upload initially targeted the previous instance’s address. Also check that SSH port 22 allows your current IP. |
| `No such file or directory` during upload | Run SCP on the Mac and check the local file path. Ubuntu cannot read the Mac’s `/Users/...` folder. |
| Key file not found | Use `~/.ssh/DataStudents.pem`, including the dot before `ssh`. |
| SSH reports an unprotected private key | On the Mac, run `chmod 400 ~/.ssh/DataStudents.pem`, then retry. |
| `Permission denied (publickey)` | Check the username is `ubuntu` and the key matches the instance’s key pair. |
| `No module named flask` | On Ubuntu, activate `venv`, then run `python -m pip install flask`. |
| Browser shows `ERR_SSL_PROTOCOL_ERROR` | Use `http://YOUR_PUBLIC_IP:5000`, not `https://`. This fixed our browser error. |
| Browser times out | Check the current public IP, port 5000’s My IP rule, and that `python app.py` is still running. |
| App only listens on `127.0.0.1` | Check the uploaded code uses `app.run(host="0.0.0.0", port=5000)`. |
| `Address already in use` | Another process is already using port 5000. Check whether this app is running in another terminal before starting another copy. |
| `Broken pipe` or SSH disconnects | Reconnect, activate the environment and check whether the app needs restarting. |
| `/favicon.ico` returns 404 | The browser requested a tab icon that our app does not provide. It does not prevent the home page from working. |

For an optional connection check, run this **on the Mac**, using port 22 for SSH or 5000 for the running Flask app:

```bash
nc -vz -G 5 YOUR_PUBLIC_IP 22
nc -vz -G 5 YOUR_PUBLIC_IP 5000
```

A successful connection means that TCP port is reachable; the browser test confirms the page itself works.

## Key definitions

| Term | Simple meaning and why we used it |
| --- | --- |
| Flask | A Python framework for building web apps. It serves our greeting page. |
| EC2 instance | A rented computer on AWS. It runs our app in the cloud. |
| Ubuntu | The operating system installed on the instance. |
| SSH | A secure way to control the instance from Mac Terminal. |
| SCP | Copies files securely between computers. We used it to upload `app.py`. |
| `.pem` key | The private key file used to prove we can access the instance. |
| Public IP | The internet address used to reach the instance. |
| Security group | AWS rules controlling which incoming connections can reach the instance. |
| Port | A numbered connection point. SSH uses 22; our app uses 5000. |
| My IP / `/32` | Allows one public IPv4 address: our current internet connection. |
| `sudo` | Runs a command with administrator permissions. We used it to install Ubuntu software. |
| `apt-get` | Ubuntu’s tool for downloading and installing system software. |
| Virtual environment / `venv` | A separate place for this app’s Python packages. |
| `pip` | Installs Python packages. We used it to install Flask. |
| `source` | Runs a file’s commands in the current terminal; here it activates `venv`. |
| Route | An address handled by the app. `/` is our home page. |
| HTTP / HTTPS | Ways browsers communicate with websites. HTTPS adds encryption; our exercise used HTTP. |
| `~` | The current user’s home folder on the computer running the command. |

## References

1. Pallets Projects. [Flask Quickstart](https://flask.palletsprojects.com/en/stable/quickstart/) — minimal app, routing, external access and development-server limitations.
2. Pallets Projects. [Flask Installation](https://flask.palletsprojects.com/en/stable/installation/) — virtual environments and installing Flask.
3. Amazon Web Services. [Security group rules for different use cases](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/security-group-rules-reference.html) — incoming connections and SSH source addresses.
4. Amazon Web Services. [Connect to your Linux instance using an SSH client](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-linux-inst-ssh.html) — connection prerequisites and SSH access.

The steps and results above record the deployment completed on 4 November 2026. Addresses and local usernames are placeholders so the guide can be reused.
