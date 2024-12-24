For Turkish version, please visit `main.tr` branch.

# Securing the SSH Connection

After setting up your server and obtaining its IP address, username (most likely **root**), and password, you can establish an SSH connection by typing:
```
ssh root@SERVER_IP
```
If you are unable to log in with **root** privileges, it might be because the server service you installed disables SSH access with **root** privileges (which is actually something we want).

After entering the command with "root" or the username provided by your server service, enter the password specified by the service to complete the SSH connection.


## Creating a User

__IF YOU CAN LOG IN WITH _root_ PRIVILEGES, YOU NEED TO COMPLETE THIS SECTION!__

After logging in to the server with root, we create a new user by entering the following commands. This user gains administrator privileges by using the **sudo** command.

```
adduser username
```
- The `adduser` command creates a user.
```
usermod -aG sudo username
```
- Adds the user named **username** to the **sudo** group. Users in the **sudo** group have administrator privileges.

After entering these commands, you will be prompted to create a password for the user. Enter your desired password.

- You can view user permissions by typing `cd nano /etc/sudoers` (optional).

To switch to the newly created user, type:
```
su username
```

## Generating an SSH Key Pair

Open the terminal on your local machine (PowerShell on Windows) and enter the following commands:
```
cd .ssh
```
- This command will take you to the `.ssh` directory.
```
ssh-keygen -t rsa -b 4096
```
- `ssh-keygen` is a command used to generate an SSH key pair.
- `-t rsa` specifies the key type.
- `-b 4096` indicates the length of the key in bits. Common bit lengths are 2048 and 4096. In some key types, the bit value is fixed, so `-b` may not be used.

After entering the command, it will ask for a name for the key. Here, we use **sshkey** as an example. You can use any name you like, but avoid using non-English characters.

Next, it will ask for a passphrase, which is an additional password for the key pair. You don’t have to set one, but since our goal is security, we will add one here.

To verify whether the key pair was generated, you can type `ls -l` while in the `.ssh` directory to list the files in detail. For more options on listing files, please research the `ls` command.
- The `.ssh` folder can be found in `C:\Users\username\` on Windows.

Two files will be created: one with a **.pub** extension and another without an extension. The file named **sshkey.pub** is your **Public Key** and can be shared with no issues. It is the one you upload to the remote system (in our case, the server).

The other file named **sshkey** is your **Private Key** and must never be shared. During authentication, the public key sends a _challenge_ to the remote device, and only the private key can provide the correct answer to that challenge, thus authenticating the connection.

To copy the newly created key to the server, we will use the `scp` command (remember, we’re still on our local machine):
```
scp sshkey.pub username@SERVER_IP:
```
- `scp` stands for **Secure Copy Protocol**, used for securely copying files.
- `sshkey.pub` is the public key from the SSH key pair you generated.
- In `username@SERVER_IP`, specify the username you created and the server’s IPv4 address.
- The trailing `:` indicates the directory on the server where the file will be copied. If nothing else is specified, the file will go to `/home/username`. By adding `:`, it places it in the user’s home directory (`~`).

When it asks for a password, enter your server’s password, and the file will be copied over.

## Defining the Key Pair and Configuring SSH

We will perform these steps on our server.

### Defining the Key Pair
First, navigate to your home directory:
```
cd ~
```
- `cd` moves you to the home directory.

List the files in your home directory:
```
ls
```
- `ls` olduğunuz dizindeki dosyaları listeler

If you see `sshkey.pub` listed, you’re on the right track, and SCP worked successfully. We now need to move this **Public Key** to the directory specified in the OS’s SSH configuration file. For most Linux distributions, that directory is `~/.ssh/authorized_keys`.
Make sure you’re still in the `~` directory. The command below creates a `.ssh` folder in `~`.
```
mkdir .ssh
```
- `mkdir` is used to create a new folder.
Now, let’s create the `authorized_keys` file inside the `.ssh` folder. This file contains the contents of your **Public Key**.
```
touch .ssh/authorized_keys
```
- `touch` is used to create files.
Next, we need to add the contents of `sshkey.pub` into the `authorized_keys` file:
```
cat sshkey.pub >> .ssh/authorized_keys
```
- `cat` displays or concatenates the contents of a file.
- `>>` appends the output from the file on the left into the file on the right. If you had used `>`, it would replace any existing contents with the new data.
If you wanted to define two **Public Keys**, you would simply list both file locations and use the `>>` operator, as shown below. This example won’t overwrite any existing content; it appends it.
```
cat ~/sshkey1.pub ~/sshkey2.pub >> .ssh/authorized_keys
```
- Since we’re in the home directory, we don’t actually need `~/` here. It’s just for illustration.

We no longer need the `sshkey.pub` file because its contents have been added to `authorized_keys`. To remove it:
```
rm sshkey.pub
```
- `rm` is used to remove a file.

### SSH Configuration
Now let’s configure SSH:
```
sudo nano /etc/ssh/sshd_config
```
- `nano` is a text editor that allows you to edit the file specified. `sshd_config` is the file that controls SSH settings.
- You could also use `vim`. For specifics on commands, please do your own research.

Lines with **#** at the start are comment lines. Removing the **#** changes them from commented to uncommented. Any default SSH settings are included as commented lines. You can modify them by removing the comment symbol and adjusting the corresponding values. Below are the lines we need to focus on (remove the **#** and set the values as shown):
```
PermitRootLogin no 
PubkeyAuthentication yes 
Password Authentication no 
AuthorizedKeysFile .ssh/authorized_keys .ssh/authorized_keys2
```
These lines indicate the following:
- Denies SSH connections for the root user.
- Enables public key authentication.
- Disables password-based authentication.
- Specifies the location and file(s) for public keys. These match what we set up above. If your distribution’s default settings already point here (and the lines are uncommented), there’s no need to change anything.

The four settings above are the most important. **The lines below usually come by default and don’t need changing.** (These examples come from Ubuntu 24.04 LTS; other distributions may vary.)

```
Include /etc/ssh/sshd_config.d/*.conf
KbdInteractiveAuthentication no
ChallengeResponseAuthentication no
UsePam yes
X11Forwarding yes
PrintMotd no 
AcceptEnv LANG LC_*
Subsystem       sftp    /usr/lib/openssh/sftp-server
```

After making these changes, press **CTRL+X** to exit. When prompted “Save modified buffer?”, press **Y**. You’ll then be asked whether you want to rename the file or change its directory. Just press **Enter** to save the file under its current name and location.

We need to restart the SSH service:
```
sudo systemctl restart sshd
```
- This restarts the SSH service. `systemctl` manages services and system components.

When asked for a password, enter the password for your user.
```
exit
```
Because we switched from root to **username** via `su username`, entering `exit` here returns us to root.
```
exit
```
Entering `exit` once more logs us out from the root session.

## Establishing the SSH Connection
We used to connect by typing `ssh username@SERVER_IP`, but now we need to use the command below:
```
ssh -i PRIVATE_KEY_FILE_PATH username@SERVER_IP
```
- In **PRIVATE_KEY_FILE_PATH**, enter the path to your private key.
    - On Windows, right-click on the private key file and select “Properties” to copy the file path, such as `C:\Users\username\.ssh`. Then add `\sshkey` to specify the actual key file. For example: `C:\Users\username\.ssh\sshkey`.

If you set a passphrase, you’ll be prompted for it. Type it in and press **Enter** to complete the SSH connection successfully.

## Advanced Settings

So far, we’ve only generated and assigned a key pair. While this is a major step for security, there are still a few more things you could do. You are not obliged to perform these additional steps, but our main objective is **to make SSH more secure**.

 ### Changing the Port (?) and Firewall Configuration
 Changing the port is not a major security enhancement. However, because bots often scan for open ports on 22 by default, using a different port might reduce visibility. First, open your SSH configuration file:
```
sudo nano /etc/ssh/sshd_config
```
Find the line `#Port 22`, remove the comment symbol **#**, and set a new port number:

```
Port 49777
```
Save and close the file.

Next, we need to configure the firewall. Since we set SSH to use **49777**, we must allow incoming connections on that port. The commands will vary based on your Linux distribution. For Ubuntu 24.04 LTS, we use `ufw`:
```
sudo ufw status
```
- This shows the ports currently allowed.

You’ll see **Port 22** allowed by default. We now allow the new port (49777):
```
sudo ufw allow 49777/tcp
```
- `allow` permits the specified port.
- We can use the `/` symbol to allow a specific protocol. The protocol specified after the `/` will be granted permission. We are allowing the TCP protocol because SSH connections work over TCP.

Next, run `sudo ufw status` again to confirm the port has been added.

Restart the SSH configuration:
```
sudo systemctl restart sshd
```

Now, remove the rule allowing **Port 22** and then reload the firewall:
```
sudo ufw delete allow 22/tcp
```
- `delete allow` removes a rule from the firewall’s list.
```
sudo ufw reload
```
- `reload` restarts the firewall.

If you try to connect again with the usual `ssh -i PRIVATE_KEY_FILE_PATH username@SERVER_IP`, it will fail because the default port is still 22. Instead, you need to specify the new port:

```
ssh -p PORT_NUMBER -i PRIVATE_KEY_FILE_PATH username@SERVER_IP
```
- Replace **PORT_NUMBER** with the port number you chose (49777).

You are free to choose any port number, but using default ports can cause conflicts and issues. We recommend the **49152 – 65535** range, known as **Dynamic and Private Ports**, which are typically not used by everyday services. For more information on ports, please do further research.
