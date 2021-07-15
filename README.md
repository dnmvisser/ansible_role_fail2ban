# ansible_role_fail2ban

Deploy and configure fail2ban on Debian.

# Role variables

All configuration file tasks use a template that generates fail2ban
configuration files from a dict:

`fail2ban_jail_local`: Dictionary that will be deployed as a single file "jail.local".

`fail2ban_actiond`: Dictionary where the first keys are the file names that
will be created under `/etc/fail2ban/action.d`.

`fail2ban_filterd`: Dictionary where the first keys are the file names that
will be created under `/etc/fail2ban/filter.d`.

# Example Playbook

```yaml

  - hosts: servers
    
    tasks:
      - name: install and configure fail2ban
        import_role:
          name: ansible_role_fail2ban
        tags: fail2ban
        vars:
          fail2ban_jail_local:
            DEFAULT:
              bantime: 86400
              maxretry: 2
            sshd:
              port: ssh
              logpath: '%(sshd_logs)s'
          fail2ban_actiond:
            nginx-block-map.local:
              Definition:
                srv_cmd: nginx
                blck_lst_reload: |
                  %(srv_cmd)s -qt; if [ $? -eq 0 ]; then
                                    %(srv_cmd)s -s reload; if [ $? -ne 0 ]; then echo 'reload failed.'; fi;
                                  fi;
          fail2ban_filterd:
            wordfence.conf:
              Definition:
                failregex: A user with IP address <HOST> has been locked out from signing in or using the password recovery form for the following reason
```
