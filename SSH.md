# Open SSH Key-based authentication

- (OpenSSH Key Management)[https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement]
- (Security of Interactive and Automated Access Management using SSH)[https://nvlpubs.nist.gov/nistpubs/ir/2015/NIST.IR.7966.pdf]
- (Installing OpenSSH)[https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=powershell]

## OpenSSH

|Code|Description|
|:-:|:-:|
|ssh-keygen -t rsa|Generate a private and public key in rsa|
|ssh-agent and ssh-add|For securely storing private keys|
|scp and sftp|To securely copy public key files during initial use of a server|

## Key Pairs

- Key pairs refer to the **public and private key** files that are used by certain authentication protocols.
- SSH Public Key Authentication uses asymmetric cryptographic algorithms to generate 2 key files - 1 private and 1 public.
- Private Key
    - Equivalent of a password, and should stay protected under all circumstances. 
    - If someone acquires your private key, they can sign in as you to any SSH server you have access to.
- Public Key
    - What is placed on the SSH server, and may be shared to any SSH server without compromising the private key.

## Install OpenSSH For Windows (PowerShell)

- Run PowerShell as an **Administrator**
- To ensure that OpenSSH is available, run the following command:
`Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'`

- The command should return the following output if neither are already installed.
```
Name  : OpenSSH.Client~~~~0.0.1.0
State : NotPresent

Name  : OpenSSH.Server~~~~0.0.1.0
State : NotPresent
```

- Then, install the server or client components as needed:
```
# Install the OpenSSH Client
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

# Install the OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

- Both commands should return the following output:
```
Path          :
Online        : True
RestartNeeded : False
```

- To start and configure OpenSSH Server for inital use, open an elevated PowerShell prompt (right click, Run as an administrator), then run the following commands to start the `sshd service`:
```
# Start the sshd service
Start-Service sshd

# OPTIONAL but recommended:
Set-Service -Name sshd -StartupType 'Automatic'

# Confirm the Firewall rule is configured. It should be created automatically by setup. Run the following to verify
if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue | Select-Object Name, Enabled)) {
    Write-Output "Firewall Rule 'OpenSSH-Server-In-TCP' does not exist, creating it..."
    New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
} else {
    Write-Output "Firewall rule 'OpenSSH-Server-In-TCP' has been created and exists."
}
```

## Connect to OpenSSH Server

- Connect OpenSSH Server with the OpenSSH client installed. (e.g., `ssh commonuser@<ip_address>`)
- `ssh domain\username@servername`

## Uninstall OpenSSH For Windows

```
# Uninstall the OpenSSH Client
Remove-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

# Uninstall the OpenSSH Server
Remove-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

## Host key generation

- By default, the sshd service is set to start manually.
- To start it each time the server is rebooted, run the following commands from an elevated PowerShell prompt on your server:
```
# Set the sshd service to be started automatically
Get-Service -Name sshd | Set-Service -StartupType Automatic

# Now start the sshd service
Start-Service sshd
```

- Since there's no user associated with the sshd service, the host keys are stored under `C:\ProgramData\ssh`.

## User Key Generation

- To use key-based authentication, need to generate public/private key pairs for your client. 
- `ssh-keygen.exe` is used to generate key files and the algorithms DSA, RSA, ECDSA, or Ed25519 can be specified.
- If no algorithm is specified, RSA is used.
- To generate key files using Ed25519, run the following command:
    - `ssh-keygen -t ed25519`
    - Output:
    ```
    Generating public/private ed25519 key pair.
    Enter file in which to save the key (C:\Users\username/.ssh/id_ed25519):
    ```
- Then, you will be prompted to use a passphrase to encrypt your private key files. 
- The passphrase can be empty but not recommended. 
- The passphrase works with the key file to provide 2-factor authentication.
- In this example, the passphrase is empty.
    - Output:
    ```
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in C:\Users\username/.ssh/id_ed25519.
    Your public key has been saved in C:\Users\username/.ssh/id_ed25519.pub.
    The key fingerprint is:
    SHA256:OIzc1yE7joL2Bzy8!gS0j8eGK7bYaH1FmF3sDuMeSj8 username@LOCAL-HOSTNAME

    The key's randomart image is:
    +--[ED25519 256]--+
    |        .        |
    |         o       |
    |    . + + .      |
    |   o B * = .     |
    |   o= B S .      |
    |   .=B O o       |
    |  + =+% o        |
    | *oo.O.E         |
    |+.o+=o. .        |
    +----[SHA256]-----+
    ```
- Now, the public/private ed25519 key pairs are in the location specified below.
- The `.pub` files are public keys, and the fiels without an extension are private keys:
    - Output:
    ```
    Mode                LastWriteTime         Length Name
    ----                -------------         ------ ----
    -a----         6/3/2021   2:55 PM            464 ed25519
    -a----         6/3/2021   2:55 PM            103 ed25519.pub
    ```
- Private key files are the equivalent of a password should be protected the same way you protect your password. 
- Use `ssh-agent` to securely store the private keys within a Windows security context, associated with your Windows account. 
- To start the `ssh-agent` service each time your computer is rebooted, and use `ssh-add` to store the private key 
- Run the following commands from an elevated PowerShell prompt on your server:
    ```
    # By default the ssh-agent service is disabled. Configure it to start automatically.
    # Make sure you're running as an Administrator.
    Get-Service ssh-agent | Set-Service -StartupType Automatic

    # Start the service
    Start-Service ssh-agent

    # This should return a status of Running
    Get-Service ssh-agent

    # Now load your key files into ssh-agent
    ssh-add $env:USERPROFILE\.ssh\id_ed25519
    ```
- Once you've added the key to the ssh-agent on your client, the ssh-agent will automatically retrieve the local private key and pass it to your SSH client.

## Deploying the public key

### Standard User

- The contents of your public key (\.ssh\id_ed25519.pub) needs to be placed on the server into a text file called `authorized_keys` in C:\Users\username\.ssh\. 
- You can copy your public key using the OpenSSH scp secure file-transfer utility, or using a PowerShell to write the key to the file.
- The example below copies the public key to the server (where "username" is replaced by your username). 
- You'll need to use the password for the user account for the server initially.
```
# Get the public key file generated previously on your client
$authorizedKey = Get-Content -Path $env:USERPROFILE\.ssh\id_ed25519.pub

# Generate the PowerShell to be run remote that will copy the public key file generated previously on your client to the authorized_keys file on your server
$remotePowershell = "powershell New-Item -Force -ItemType Directory -Path $env:USERPROFILE\.ssh; Add-Content -Force -Path $env:USERPROFILE\.ssh\authorized_keys-Value '$authorizedKey'"

# Connect to your server and run the PowerShell using the $remotePowerShell variable
ssh username@domain1@contoso.com $remotePowershell
```





