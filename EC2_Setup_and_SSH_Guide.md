# AWS EC2: Setup and SSH Connection from a Mac

**Date:** 3 November 2026  
**Scope:** Create an Ubuntu EC2 instance and establish an SSH connection using macOS Terminal.

## Introduction

Amazon Web Services (AWS) provides cloud computing services. Amazon Elastic Compute Cloud (EC2) allows users to create virtual servers, called **instances**, with a selected operating system and computing resources. This makes it possible to work on a remote server without purchasing and maintaining the physical hardware [1].

In this exercise, the Mac is the **local computer**, while the Ubuntu EC2 instance is the **remote computer**. Secure Shell (SSH) provides a secure connection between them. After connecting, commands entered in the same Terminal window run on the remote Ubuntu instance [3].

### Key terms

| Term | Meaning |
|---|---|
| AMI | Amazon Machine Image: a template containing the operating system and associated software. |
| Instance type | The configuration of computing resources available to the virtual server. |
| Key pair | A public and private key used for authentication. The private `.pem` file stays on the Mac. |
| Security group | A virtual firewall controlling permitted network traffic. |
| VPC and subnet | The virtual network and network section in which the instance operates. |
| EBS volume | Persistent block storage used by the instance, including its operating-system disk. |

Definitions follow AWS documentation [1, 2].

## 1. Open EC2 and choose the region

Sign in to the AWS Management Console, search for **EC2**, and open the service. Set the region to **Europe (Frankfurt), `eu-central-1`**.

**Why:** This keeps the exercise in the same region as the original class setup.

## 2. Name the instance

Select **Launch instances**. Under **Name and tags**, enter:

```text
Data-606-Sabahi
```

**Why:** A descriptive name makes the instance easier to identify in the console.

## 3. Select Ubuntu and the instance type

Under **Application and OS Images**, select the official **Ubuntu Server 26.04 LTS** image with **64-bit (x86)** architecture. Under **Instance type**, choose **`t3.micro`**.

**Why:** Ubuntu provides the operating system; `t3.micro` provides the computing configuration used in this exercise. The instance runs independently of the Mac, so a Mac with an ARM processor can connect to an x86 EC2 instance.

**Recorded result:** The successful login identified Ubuntu **26.04.1 LTS** on `x86_64`.

## 4. Select the key pair

Under **Key pair (login)**, select **`DataStudents`**, matching the existing private key:

```text
DataStudents.pem
```

**Why:** The matching private key proves your identity when connecting. A similarly named but different key will not work.

For a first-time setup, if the matching key pair does not exist, choose **Create new key pair**, follow the class requirements and download the private key in `.pem` format. Use the downloaded filename consistently in the later commands.

Keep the private key secure; do not include its contents in documentation or commit it to GitHub.

## 5. Configure the network and security group

Under **Network settings**, select the class VPC and subnet and enable **Auto-assign public IP**. Create or select the security group **`sabahij-sg`** and ensure it is attached to the instance.

Configure this inbound rule:

| Setting | Selection |
|---|---|
| Type | SSH |
| Protocol | TCP |
| Port | 22 |
| Source | My IP |

**Why:** This permits an SSH connection from your current public IP address. A source such as `203.0.113.10/32` represents one IPv4 address; this is an illustrative example, not a value to copy. Select **My IP** to obtain your own address [3, 4].

## 6. Review storage and launch

Under **Configure storage**, check the root volume settings against the class requirements. The exact configured size and volume type were not captured in this exercise, so they are not specified here.

Review the configuration and displayed charges, select **1 instance**, and choose **Launch instance**. Free usage depends on account eligibility and available benefits; do not assume this configuration is free [2].

**Expected result:** A launch confirmation and a new instance ID.

## 7. Check readiness and copy the address

Open the instance details. Wait for **Running** and all status checks to pass. In this exercise, AWS displayed **3/3 checks passed**.

Choose **Connect → In SSH client** and locate the connection information. Copy the instance's **Public IPv4 DNS** [3].

The address used successfully in this exercise was:

```text
ec2-18-192-125-230.eu-central-1.compute.amazonaws.com
```

For another instance, substitute its current address throughout this guide.

## 8. Navigate to the key on the Mac

Open **Terminal on the Mac**. Run each command separately.

### Go to the home directory

```bash
cd ~
```

`cd` changes directory; `~` means the current user's home directory.

### Check the current location

```bash
pwd
```

Recorded output:

```text
/Users/sabahij
```

### Find the SSH folder

```bash
ls -a
```

`ls` lists directory contents. `-a` includes hidden items, including `.ssh`.

### Prepare the folder and key, if needed

For a first-time setup only, create the folder and move the downloaded key:

```bash
mkdir -p ~/.ssh
mv -i ~/Downloads/DataStudents.pem ~/.ssh/
```

`mkdir -p` creates the folder if it does not exist. `mv -i` moves the key and asks before overwriting an existing file. Skip the move if the correct key is already in `.ssh`.

### Enter the folder and check the key

```bash
cd ~/.ssh
ls -l DataStudents.pem
```

Recorded output:

```text
-r--------@ 1 sabahij staff 1670 3 Nov 12:02 DataStudents.pem
```

`-r--------` confirms owner-only read permission. At this point, the prompt still identifies the Mac:

```text
sabahij@MacBookAir .ssh %
```

## 9. Connect to the Ubuntu instance

From the `.ssh` directory, run:

```bash
ssh -i DataStudents.pem ubuntu@ec2-18-192-125-230.eu-central-1.compute.amazonaws.com
```

| Command element | Explanation |
|---|---|
| `ssh` | Opens the secure connection. |
| `-i DataStudents.pem` | Selects the private key in the current directory. |
| `ubuntu` | Uses the Ubuntu image's default login account [4]. |
| Address after `@` | Identifies the remote EC2 instance. |

From any other directory, use the full key path instead:

```bash
ssh -i ~/.ssh/DataStudents.pem ubuntu@ec2-18-192-125-230.eu-central-1.compute.amazonaws.com
```

On first connection, SSH displays a host fingerprint and asks whether to continue. This identifies the server, rather than your login key. AWS describes comparing it with the instance system log via **Actions → Monitor and troubleshoot → Get system log**. After checking that it matches, type `yes` and press **Enter** [2].

SSH then records the host in `known_hosts`.

## 10. Recognise a successful connection

The completed exercise returned:

```text
Welcome to Ubuntu 26.04.1 LTS
```

The prompt changed to:

```text
ubuntu@ip-172-31-27-209:~$
```

**Meaning:** The SSH connection succeeded. Commands entered at this prompt run on the EC2 instance. The private address in the hostname can differ from the public address used to connect.

Optional verification commands are:

```bash
whoami
hostname
cat /etc/os-release
```

These show the remote user, hostname and operating system. They are additional checks; the welcome message and changed prompt already confirmed the recorded connection.

**End of exercise:** Leave the session connected for the next class activity. This guide ends at SSH access.

## Troubleshooting

| Issue | Explanation and action |
|---|---|
| `Identity file ... not accessible` | Check the filename and path. Use `~/.ssh/`, including the dot. |
| `Operation timed out` | Check the instance state, status checks, current public address and attached security group's SSH rule. Set its source to your current **My IP** and save. |
| Blank window during a connection attempt | The connection may still be pending. Press **Control + C** to cancel the attempt. |
| IPv4 CIDR cannot replace an existing IPv6 rule | Delete the incorrect rule and create a new **SSH → My IP** rule, then save. |
| Error involving a referenced security group | Use your current IP address as the source for this Mac-to-instance connection. |
| `Permission denied (publickey)` | Check that `ubuntu` is the correct username and the private key matches the selected key pair. |
| Private key permissions too open | Run `chmod 400 ~/.ssh/DataStudents.pem`. |

The first attempts in this exercise timed out. After the inbound security group rules were modified, the same SSH command succeeded. This sequence supports checking network access before changing a correctly located key. AWS provides further troubleshooting guidance [4].

An optional command limits the connection wait to ten seconds; it does not repair a network problem:

```bash
ssh -o ConnectTimeout=10 -i ~/.ssh/DataStudents.pem ubuntu@ec2-18-192-125-230.eu-central-1.compute.amazonaws.com
```

## Recorded configuration and outcome

| Item | Recorded value |
|---|---|
| Instance name | `Data-606-Sabahi` |
| Instance ID | `i-0543d5a3c4beefdc1` |
| Region | Frankfurt, `eu-central-1` |
| Operating system | Ubuntu 26.04.1 LTS, x86_64 |
| Instance type | `t3.micro` |
| Key pair | `DataStudents` |
| Local private-key path | `/Users/sabahij/.ssh/DataStudents.pem` |
| Security group | `sabahij-sg` — `sg-0e351ce58d39a1f5e` |
| Inbound access | SSH, TCP 22; My IP selected during setup |
| VPC | `vpc-001148a5b29d31b39` |
| Subnet | `subnet-0532036092904c0c4` |
| Public IPv4 address | `18.192.125.230` |
| Private IPv4 address | `172.31.27.209` |
| Readiness | Running; 3/3 checks passed |
| Outcome | SSH login successful |

Configuration and results are based on the supplied screenshots and terminal output. The AMI ID and exact EBS volume configuration were not captured. Addresses and resource IDs are specific to this exercise and should be replaced when recreating the setup.

## References

[1] Amazon Web Services (n.d.) *What is Amazon EC2?* Available at: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html (Accessed: 3 November 2026).

[2] Amazon Web Services (n.d.) *Get started with Amazon EC2*. Available at: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html (Accessed: 3 November 2026).

[3] Amazon Web Services (n.d.) *Connect to your Linux instance using SSH*. Available at: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-to-linux-instance.html (Accessed: 3 November 2026).

[4] Amazon Web Services (n.d.) *Troubleshoot issues connecting to your Amazon EC2 Linux instance*. Available at: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstancesConnecting.html (Accessed: 3 November 2026).
