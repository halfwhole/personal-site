# readme

## Installation

Ensure that `rbenv` is set up, then install the `jekyll` gem:

```sh
$ gem install jekyll
```

## Serving Locally

``` sh
$ jekyll serve
```

## Deployment

Create a new file `production.ini`, making sure the user has `sudo` privileges:

``` ini
[remote]
<your.ip.address.here> ansible_user=<user>
```

Deploy the site to your remote server:

``` sh
$ ansible-playbook playbook.yml -i production.ini
```

## LaTeX

To write LaTeX, put this in the beginning:

``` html
<script src="/public/js/jquery-1.11.1.min.js"></script>
<script src="/public/js/katex.min.js"></script>
<link rel="stylesheet" href="/public/css/katex.min.css">
```

Put this at the end:

``` html
<script src="/public/js/katex_render.js"></script>
```

Write LaTeX as such:

``` html
<script type="math/tex">\frac{41}{50}</script>
```

## Admin Details

This site is currently being run on a CentOS machine.

To start/stop serving the site:

``` sh
$ sudo systemctl start nginx
$ sudo systemctl stop nginx
```

### CertBot

SSL/TLS certificates are configured with CertBot.

To setup new certificates for the website:

```sh
$ sudo certbot --nginx -d <site.com> [-d <www.site.com>]
```

To renew existing certificates for the website:

```sh
$ sudo certbot renew
```

There's also an existing problem whereby the configurations for CertBot in `nginx.conf` can be overriden,
which I believe (?) happens every time the ansible playbook is run.
To fix this issue, we need to re-setup CertBot for nginx afterwards.

