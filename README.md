# readme

## Serving Locally

``` sh
jekyll serve
```

## Deployment

Create a new file `production.ini`, making sure the user has root privileges:

``` ini
[remote]
<your.ip.address.here> ansible_user=<user>
```

Deploy the site to your remote server:

``` sh
$ ansible-playbook playbook.yml -i production.ini
```
