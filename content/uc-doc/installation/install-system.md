---
title: Installing the System
---

## Requirements

- Memory: 2 GiB of memory is a tight minimum
- Storage: 2.5 GiB of storage is a very tight minimum, 8 GiB is comfortable

## Procedure

To install the Unified Communication use case in an all-in-one setup, do the following steps:

1. Install a Debian 11 Bullseye system with a default locale with an UTF-8 charset.
2. Run the following commands as root on the Debian system to provision sudo, git and Ansible:

   ```shell
   apt update
   apt install -yq sudo git ansible curl
   ```

3. Extract the Wazo Platform installer

   ```shell
   git clone https://github.com/wazo-platform/wazo-ansible.git
   cd wazo-ansible
   ```

4. (optional) By default, Wazo Platform will install the development version. To install the latest
   stable version

   ```shell
   ansible_tag=wazo-$(curl https://mirror.wazo.community/version/stable)
   git checkout $ansible_tag
   ```

5. Install the Wazo Platform installer dependency

   ```shell
   ansible-galaxy install -r requirements-postgresql.yml
   ```

6. Edit the Ansible inventory in `inventories/uc-engine` to add your preferences and passwords. The
   various variables that can be customized are described at
   <https://github.com/wazo-platform/wazo-ansible/blob/master/README.md#variables>.

   By default, Wazo Platform will install the development version. To install the latest stable
   version, activate the following settings in your inventory:

   ```ini
   [uc_engine:vars]
   wazo_distribution = pelican-bullseye
   wazo_distribution_upgrade = pelican-bullseye
   ```

   If you want to install the web user interface, activate the following in your inventory:

   ```ini
   [uc_ui:children]
   uc_engine_host
   ```

   To create the `root` account at installation time and be able to use the web user interface and
   REST APIs, you need to add the following variables:

   ```ini
   [uc_engine:vars]
   engine_api_configure_wizard = true
   engine_api_root_password = ****
   ```

7. Launch the installation by running the following command:

   ```shell
   ansible-playbook -i inventories/uc-engine uc-engine.yml
   ```

## Use the REST API

You may now use the REST API from outside your system (here `wazo.example.com`).

1. Get an authentication token for 1 hour:

   Using the `api_client_name` and `api_client_password` you defined in your inventory, you can
   execute from the Debian system:

   ```shell
   wazo-auth-cli token create --auth-user <api_client_name> --auth-password <api_client_password>
   ```

   Or with curl from anywhere:

   ```shell
   curl -k -X POST -u <api_client_name>:<api_client_password> -H 'Content-Type: application/json' -d '{"expiration": 3600}' https://wazo.example.com/api/auth/0.1/token
   ```

2. Use any REST API you want, for example, to list the telephony users configured on the system:

   Note: You should replace the following values:

   - `my-token` with the authentication token

   ```shell
   curl -k -X GET -H 'X-Auth-Token: <my-token>' -H 'Content-Type: application/json' -d '{"firstname": "user1"}' https://wazo.example.com/api/confd/1.1/users
   ```

   Note: the token that you have now only has permissions for configuration REST API (wazo-confd).

## Optional post-install steps

You may now follow the [optional post-install steps](/uc-doc/installation/postinstall).
