# WSL 1 with Docker on VirtualBox

## Introduction
This script is a way to manage in a single command line the Docker on VirtualBox.

This is the most effective way I have found to use docker with Windows Subsystem for Linux 1 without losing performance and without using Hyper-V (docker for Windows).

The advantage of this is to use the docker in a native linux file system and the ability to work with docker compose.

## Step-to-step

Basically:

1. Windows Subsystem for Linux 1 installed and correctly configured;
2. Any linux distro installed on VirtualBox with following details:
    - Create a [**shared folder**](#shared-folder) between your project folder from host to /mnt/dev (or another dir) in guest;
    - [**NAT**](#nat) configured with SSH port forwarding and other services you will use (nginx, mysql, etc);
    - Booting in [**multi-user**](#multi-user) mode (X11 is not required);
    - SSH server with [**public key authentication**](#public-key-authentication);
    - [**GRUB**](#grub) configured with GRUB_TIMEOUT to 0 seconds (immediately boot);
    - [**fstab**](#fstab) configured with uid=1000 and gid=1000 (do not use automount option from shared folder);
    - [**Docker**](#docker) installed and correctly configured.

 ### [Shared Folder](#shared-folder)

 On VirtualBox:
 
 - Configure
    - Shared Folders
        - Add a new shared folder

 Example:

- **Folder Path**: C:\Users\Username\projects
- **Folder Name**: Projects
- **Read-only** and **Auto-mount**: uncheck all
- **Mount point**: /mnt/projects

### [NAT](#nat)

On VirtualBox:

- Configure
    - Network
        - Advanced
            - Port Forwarding
                - Add a new port forwarding rule 

Setup a SSH forward port connection:

| Name | Protocol | Host IP   | Host Port | Guest IP  | Guest Port |
| ---  | ---      | ---       | ---       | ---       | ---        |
|  SSH |   TCP    | 127.0.0.1 |  22       | 10.0.2.15 | 22         |

If port 22 is in use on host, setup another port like 2201, 2202, etc.

### [Multi-User](#multi-user)

If you installed linux without X then you can skip this part.

Why disable the X? Because we won't use it unless you want to use it for other things.

To disable GDM/SDDM/LightDM, boot your linux vm and type:
```
systemctl set-default multi-user.target
```

### [Public Key Authentication](#public-key-authentication)

In your WSL terminal, type:
```
ssh-keygen
```

Press enter for all questions, and copy your public key from **~/.ssh/id_rsa.pub**:
```
cat ~/.ssh/id_rsa.pub
```

Open VirtualBox and boot your linux vm. Run the same command above (**ssh-keygen**).

Paste your **id_rsa.pub** content with nano or vim on **~/.ssh/authorized_keys** and save.

Enable SSH service:
```
systemctl start ssh
```

Test SSH connection from WSL terminal: 
```
ssh user_of_linux_vm@127.0.0.1 -p 22
```

Port 22 is the host port configured on the NAT. Change here if you put another port.

If it works, you can log out of the ssh session. 

To finish, enable SSH autostart service on linux vm:
```
systemctl enable ssh
```

### [GRUB](#grub)

Edit **/etc/default/grub** on linux vm and set *GRUB_TIMEOUT* to 0:
```
GRUB_TIMEOUT=0
```

Save and exit. Run:
```
update-grub
```

### [FSTAB](#fstab)

Edit **/etc/fstab** on linux vm and paste this at the end of the file:
```
Folder_Name	Mount_point	vboxsf	rw,uid=1000,gid=1000	0	0
```

Where **Folder_Name** and **Mount_point** are your configurations in shared folders.

Don't forget to add your user of linux vm to **vboxsf** group.

### [DOCKER](#docker)

After install docker on linux vm, add your user to docker group:
```
gpasswd -a user_of_linux_vm docker
```

### Box

Put box script on WSL bin path (example: /usr/local/bin) and change the script execution permission to 755:
```
sudo chmod 755 /usr/local/bin/box
```

Create a **~/.box_list** file and configure with your linux vm data. For example:
```
# vm to use with docker
LINUXBOX_USER=dev
LINUXBOX_HOST=127.0.0.1   
LINUXBOX_PORT=2200
```

**LINUXBOX** is your vm name. 

Save and exit. To use:
```
box LinuxBox docker run hello-world
```

### box_list

Configuration file with some settings.

This file accept options with suffix;

- `_USER` : user of linux vm (**REQUIRED**)
- `_HOST` : host (**REQUIRED**)
- `_PORT` : port (OPTIONAL | DEFAULT: 22)
- `_X11`  : enable X11 apps (OPTIONAL | DEFAULT: false)

For example, assuming two linux vm with names like **Debian** and **Kali**.

```
# ~/.box_list
DEBIAN_USER=dev
DEBIAN_HOST=127.0.0.1
DEBIAN_PORT=2200
DEBIAN_X11=true

KALI_USER=root
KALI_HOST=192.168.25.111
```