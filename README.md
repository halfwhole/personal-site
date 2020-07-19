# readme

Create a new file `production.ini`:

``` ini
[remote]
<your.ip.address.here> ansible_user=<user>
```

Deploy the site to your remote server:

``` sh
$ ansible-playbook playbook.yml production.ini
```
