# jitsi_meet

An Ansible role to install _Jitsi Meet_.

## Introduction

This is an [Ansible](https://docs.ansible.com/ansible/latest/index.html) role that installs [_Jitsi Meet_](https://jitsi.org/jitsi-meet/) with [nginx](https://nginx.org/) as TLS terminating proxy and (optionally) a [Let's Encrypt certificate](https://letsencrypt.org/) (via [certbot](https://certbot.eff.org/)) on [Ubuntu Bionic (18.04)](http://releases.ubuntu.com/18.04/).

## Requirements

* A domain must point to your server in order to use Let's Encrypt
* Your firewall must allow ports 80/tcp, 443/tcp, 4443/tcp, 10000/udp
  * If your server is behind a NAT, then make sure to forward these ports.

## Role Variables

- `apt_mirror`: On Ubuntu, _universe_ must be enabled. This variable should indicate your system mirror. Defaults to `http://archive.ubuntu.com/ubuntu`
- `jitsi_domain`: Under which domain will Jitsi be accessible. Must be a domain name if you intend to use Let's Encrypt. Can be an IP otherwise. Defaults to `{{ inventory_hostname }}`.
- `certbot_enabled`: Whether to install certbot and request a certificate for `{{ jitsi_domain }}`. Defaults to `false`.
- `certbot_admin_email`: Which email address to register for Let's Encrypt. Required if `certbot_enabled=true`. The email should exist. No default value.
- `jitsi_nat`: Whether you're running _jitsi meet_ behind a NAT. Defaults to `false`. If enabled, you must set `jitsi_nat_local_ip` and `jitsi_nat_public_ip`.
- `jitsi_nat_public_ip`: The public IP of your _jitsi meet_ host. Defaults to the IPv4 reported by [ipify](https://www.ipify.org/).
- `jitsi_nat_private_ip`: The private IP of your _jitsi meet_ host. Defaults to the IPv4 that Ansible considers to be the default for the host.
- `nginx_server_names_hash_bucket_size`: The `server_names_hash_bucket_size` of nginx. Will be declared in the global `nginx.conf` if `nginx_modify_server_names_hash_bucket_size` is set to `true`. Defaults to `64`.
- `nginx_modify_server_names_hash_bucket_size`: Whether to change the value of `server_names_hash_bucket_size` in the global `nginx.conf` file. Defaults to `true`.

Also look at [geerlingguy/ansible-role-certbot/.../defaults/main.yml](https://github.com/geerlingguy/ansible-role-certbot/blob/master/defaults/main.yml) for further configuration settings that are related to _certbot_.

## Dependencies

Depends on the [`geerlingguy.certbot` Ansible role](https://github.com/geerlingguy/ansible-role-certbot) for the Let's Encrypt / certbot tasks:

```bash
ansible-galaxy install geerlingguy.certbot
```

## Quickstart

[Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html):

```bash
# on macOS with Homebrew
brew install ansible

# on Debian
sudo apt update
sudo apt install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible

# or via Python / pip
sudo python3 -m pip install ansible
```

Setup a new server with [Ubuntu 18.04](http://releases.ubuntu.com/18.04/) or get one on [Digital Ocean](https://m.do.co/c/50be0310e60b), [Vultr](https://www.vultr.com/?ref=8496145-6G), [Hetzner Cloud](https://www.hetzner.com/cloud), [Cloudscale](https://www.cloudscale.ch/), Azure, Google Cloud, AWS, ...

Make sure you can [login via your SSH key](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server):

```bash
ssh-copy-id ubuntu@my-jitsi-server.com
ssh ubuntu@my-jitsi-server.com
```

Create an inventory file:

```ini
# jitsi.ini
[jitsi]
my-jitsi-server.com jitsi_domain=my-jitsi-server.com certbot_admin_email=admin@my-jitsi-server.com

[jitsi:vars]
ansible_user=ubuntu
ansible_become=yes # set to `no` if you log in via root
apt_mirror=http://archive.ubuntu.com/ubuntu # change to the mirror you already use
certbot_enabled=yes
jitsi_nat=no # turn on if your server is behind a NAT.
```

Create a playbook file:

```yaml
# jitsi.yml
- hosts: jitsi
  roles:
      - cimnine.jitsi_meet
```

Install the required dependencies:

```bash
ansible-galaxy install cimnine.jitsi_meet
ansible-galaxy install geerlingguy.certbot
```

Run the playbook file on the inventory:

```bash
# if `sudo` on your server does not require a password:
ansible-playbook -i jitsi.ini jitsi.yml

# or if `sudo` on your server requires a password:
ansible-playbook -K -i jitsi.ini jitsi.yml
```

## Configuration

Jitsi-meet can be configured from this ansible-role. To do so, the
`config.js` from upstream will be replaced by a file managed by this
role.

Enable this behaviour with the setting: `use_custom_jitsi_config_vars:
yes`. 

Then, set all required config variables. The defaults can be found in
file `defaults/main.yml` of the role. All variables must be provided,
not just those that you want to override! Pay attention to empty
variables such as `analytics: {}`: due to the way Jitsi works, this has
to be an empty object, and cannot be null.

All settings, their values, usage and effect are documented
in [jitsi-meet config.js](https://github.com/jitsi/jitsi-meet/blob/stable/jitsi-meet_4101/config.js).

An incomplete example is:
```
jitsi_config:
  hosts:
    domain: "{{ jitsi_domain }}"
    muc: "conference.{{ jitsi_domain }}"
  bosh: "//{{ jitsi_domain }}/http-bind"
  clientNode: "http://jitsi.org/jitsimeet"
  testing:
    enableFirefoxMulticast: true # Defaults to false
```

NOTE: the requirements of config.js may change at any moment when
updating jitsi-meet, upstream is not very comunnicatative about this.
Check with any CHANGELOG, and announcements about new required values
before you update.

NOTE: the structure is case-sensitive and follows the exact variable
settings in config.js. So `webrctIceUdpDisable` is *not* the same as
`WebRTCIceUDPDisable`. Jitsi is inconsistent in its naming of variables
(e.g. It mixes `URL` and `Url` at random), so pay attention to the exact name.

## Uninstall

The following commands help you to remove the installation.
They might not completely remove every file, but it's enough to start again should you messed up something.

```bash
systemctl stop jitsi-videobridge
systemctl disable jitsi-videobridge
apt-get purge -y jigasi jitsi-meet jitsi-meet-web-config jitsi-meet-prosody jitsi-meet-web jicofo jitsi-videobridge

systemctl stop nginx
systemctl disable nginx
apt-get purge -y nginx nginx-common nginx-full

apt purge certbot

rm -rf /etc/jitsi /etc/nginx /etc/letsencrypt

crontab -e -u root

reboot
```

## License

MIT
