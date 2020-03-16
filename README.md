# jitsi-meet
=========

Install jitsi-meet with nginx and (optionally) a Let's Encrypt certificate (via certbot) on Ubuntu Bionic through Ansible.

## Requirements

* A domain must point to your server in order to use Let's Encrypt
* Your firewall must allow ports 80/tcp, 443/tcp, 4443/tcp, 10000/udp

## Role Variables

- `apt_mirror`: On Ubuntu, _universe_ must be enabled. This variable should indicate your system mirror. Defaults to `http://archive.ubuntu.com/ubuntu`
- `jitsi_domain`: Under which domain will Jitsi be accessible. Must be a domain name if you intend to use Let's Encrypt. Can be an IP otherwise. Defaults to `{{ inventory_hostname }}`.
- `certbot_enabled`: Whether to install certbot and request a certificate for `{{ jitsi_domain }}`. Defaults to `false`.
- `certbot_admin_email`: Which email address to register for Let's Encrypt. Required if `certbot_enabled=true`. The email should exist. No default value.
- `jitsi_nat`: Whether you're running _jitsi meet_ behind a NAT. Defaults to `false`. If enabled, you must set `jitsi_nat_local_ip` and `jitsi_nat_public_ip`.
- `jitsi_nat_public_ip`: The public IP of your _jitsi meet_ host. Defaults to the IPv4 reported by [ipify](https://www.ipify.org/).
- `jitsi_nat_private_ip`: The private IP of your _jitsi meet_ host. Defaults to the IPv4 that Ansible considers to be the default for the host.

See https://github.com/geerlingguy/ansible-role-certbot/blob/master/defaults/main.yml for further _certbot_ related configuration settings.

## Dependencies

Depends on `geerlingguy.certbot` for the Let's Encrypt / certbot tasks:

```bash
ansible-galaxy install geerlingguy.certbot
```

## Quickstart

Create an inventory:

```ini
# jitsi.ini
[jitsi]
my-jitsi-server.com certbot_admin_email=admin@my-jitsi-server.com

[jitsi:vars]
ansible_become=yes
certbot_enabled=true
```

Create a playbook:

```yaml
# jitsi.yml
- hosts: jitsi
  vars:
    apt_mirror: http://archive.ubuntu.com/ubuntu
  roles:
      - jitsi
```

Run these commands:

```bash
ansible-galaxy install cimnine.jisti-meet
ansible-galaxy install geerlingguy.certbot

# if `sudo` on your server does not require a password:
ansible-playbook -i jitsi.ini jitsi.yml

# or if `sudo` on your server requires a password:
ansible-playbook -K -i jitsi.ini jitsi.yml
```

## Uninstall

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
