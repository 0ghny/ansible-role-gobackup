# ansible-role-gobackup
> Configure backups using GoBackup

This role install and deploy [gobackup](https://gobackup.github.io/) with configuration and optionally also installs and start it as a SystemD service.

## Role Variables

Available variables are listed below, along with defautl values (see [defaults/main.yml](./defaults/main.yml))

```shell
gobackup_dir: "/etc/gobackup"
# if you want to change the owner and group of the docker_stacks_dir
#gobackup_owner: "{{ ansible_user }}"
# if you want to change the owner and group of the docker_stacks_dir
#gobackup_group: "{{ ansible_user }}"
gobackup_bin_path: "/usr/local/bin/gobackup"
gobackup_tmp_dir: "/tmp"

# use 'vx.x.x' to install specific version
gobackup_version: 2.11.1
gobackup_platform: linux
gobackup_arch: amd64

gobackup_systemd: true

gobackup_web_address: "127.0.0.1"
gobackup_web_port: 2703
gobackup_web_username: "gobackup"
gobackup_web_password: "gobackup"
# gobackup_models
```

`gobackup_version` is the version to install (without the `v`): 
By default it installs the binary for plaform `linux` and `amd64` arch, but you can customize that using `gobackup_platform` and `gobackup_arch` variables.

`gobackup_dir` is the directory where gobackup configuration will reside, the `gobackup.yml` file plus other run files like `logs`, `pid`, etc.
If you wanna change the default user used by ansible by another user present in the box, you can use both `gobackup_owner` and `gobackup_group` to make sure that directories are chown to this.
`gobackup_bin_path` is where the downloaded binary will be installed/copied to, defaults to `/usr/local/bin/gobackup`.

**GoBackup Webserver** is enabled by default when using `gobackup_systemd` to `true` and it will leave the [WebUI](https://gobackup.github.io/#web-ui) to connects to. It can be **password** protected by setting up `gobackup_web_username` and `gobackup_web_password`.
By default it listens to `127.0.0.1` on port `2703`. Both can be configured using `gobackup_web_address` and `gobackup_web_port`.

`gobackup_models` is the section inside `gobackup.yml` file that contains the [gobackup models](https://gobackup.github.io/configuration#modelconfig)

## Example

```yaml
- name: Configure backups using GoBackup
  ansible.builtin.import_role:
  name: gobackup
  vars:
    gobackup_systemd: true
    gobackup_models: |
        my_app:
            schedule:
                every: "1day"
                at: "10:30"
            compress_with:
                type: tgz
            storages:
                local_a:
                    type: local
                    path: /data/backups/my_app
            archive:
                # archive multiple files / directoies
                includes:
                    - /home/git/.ssh/
                    - /etc/mysql/my.conf
                    - /etc/nginx/nginx.conf
                    - /etc/nginx/conf.d
                    - /etc/redis/redis.conf
                    - /etc/logrotate.d/
                # path will exclude
                excludes:
                    - /home/ubuntu/.ssh/known_hosts
                    - /etc/logrotate.d/syslog
            before_script: |
                # Stop database service
                systemctl stop mysql
            after_script: |
                # Start database service
                systemctl start mysql
            databases:
                mysql:
                    type: mysql
                    host: localhost
                    database: my_app_database
                    username: user
```
