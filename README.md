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

## Admin Details

To start/stop serving the site:

``` sh
$ sudo systemctl start nginx
$ sudo systemctl stop nginx
```

## LaTeX

Put this in the beginning:

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
