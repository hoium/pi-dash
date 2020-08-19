# pi-dash

Provision Raspberry Pi to Display a Dashboard

## Requirements

* [Ansible](http://www.ansible.com/)
* [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
* [Raspberry Pi OS](https://www.raspberrypi.org/downloads/raspberry-pi-os/)


## Setup Raspberry PI

**Install Raspbian OS**

* Format your SD using Disk Utility ~ Format MS-DOS (FAT)
* Download [Raspberry Pi OS](https://www.raspberrypi.org/downloads/raspberry-pi-os/)
* Unzip the download
* Burn the image to your SD Card Using [Etcher](https://etcher.io/)
* Insert your SD and turn on your Pi.

**Enable SSH Access**

* Open your Terminal
* Run `sudo raspi-config`
* Select `5 Interfacing Options`
* Select `P2 SSH`
* Select Yes and Press Enter
* Select `5 Interfacing Options`
* Select `P3 VNC`
* Select Yes and Press Enter
* Reboot your Pi

**Retrieve Pi's IP Address**

* Connect your Pi to the network through ethernet or WIFI.
* Open your Terminal
* Run the `ifconfig` command
* Make note of the IP Address under `eth0`

## Installation

Setup the ansible script.

```
cd ansible-pi
```

Edit the `hosts` file by replacing `host-ip` with your IP.

Update variables in `vars/all.yml`:

* `host_name`
* `host_ip`
* `ssh_pub_key`

Update variables in `vars/secrets.yml`:

- `user_password`
- `pi_password`

Encyrpt `vars/secrets.yml` using [Ansible Vault](#Ansible-Vault):

`ansible-vault edit vars/secrets.yml`

Deploy using [Ansible](http://www.ansible.com).

```
ansible-playbook playbook.yml -i hosts --ask-pass --become -c paramiko
```
Pi default password `raspberry`.

Let Ansible do it's Magic :)

## What does this playbook do?
1. Config
    - Generate locale and set it to en_US.UTF-8
    - Update Timezone to America/New York
    - Set hostname to the defined value - `host_name`
    - Add `127.0.0.1 host_name` to hosts file
    - Disables Screen Sleep

2. SSH
    - Removes Root SSH Configuration
    - Updates `sshd_config` Parameters

3. VNC
    - Enables VNC Server
    - Enables VNC Command
    - Enables VNC Port
    - Enables VNC Server Width
    - Enables VNC Server Height
    - Enables VNC Server Depth

4. APT
    - Installs Packages - `apt_install_packages`
    - Removes Packages - `apt_remove_packages`
    - APT Full System Update
    - Reboot if required

5. User
    - Creates a new user - `user_name`
    - Adds user to groups - `user_groups`
    - Uploads `ssh_pub_key` to authorized_keys
    - * If your key is not located in `~/.ssh/id_rsa.pub` update `ssh_pub_key`
    - Enables password less sudo
    - Sets `user_name` password to `user_password`
    - Changes Pi's password to - `pi_password`
    - Have Ansible to use new `pi_password`
    - Changes AutoLogin to `user_name`
    - Restarts Pi
    - Waits for SSH to come up

6. Chromium
    - Launches Chromium on Boot
    - * Full Screen, Incognito, Geckoboard URL
    - Restarts Pi
    - Waits for SSH to come up


## Ansible-Vault

To use `ansible-vault` you must have the file `~/.vault_pass.txt` on your machine with the vault password inside. The password is stored in 1Password Vault.

`user_password` and `pi_password` are stored in `vars/secrets.yml`.

Encrypt:  
` ansible-vault edit vars/secrets.yml`

Decrypt:  
`ansible-vault decrypt vars/secrets.yml`

Edit or View:  
`ansible-vault edit vars/secrets.yml`

## Using Playbook after Initial Setup

Add Pi to `~/.ssh/config`

```
Host $IP $Hostname
    HostName $IP
    User ops
    IdentityFile ~/.ssh/$ssh-key
```
Change `remote_user` to `user_name` on playbook.yml

Edit `main.yml` to include the tasks you want to run

Run the playbook:
`ansible-playbook playbook.yml -i hosts`
