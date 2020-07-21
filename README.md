# readme

## Serving Locally

``` sh
jekyll serve
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

Put this in the beginning:

``` html
<script src="/assets/js/jquery-1.11.1.min.js"></script>
<script src="/assets/js/katex.min.js"></script>
<link rel="stylesheet" href="/assets/css/katex.min.css">
```

Put this at the end:

``` html
<script src="/assets/js/katex_render.js"></script>
```

Write LaTeX as such:

``` html
<script type="math/tex">\frac{41}{50}</script>
```
