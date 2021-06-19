# StevenHalla.com

My personal website and blog


### Serve Locally

```bash
bundle exec jekyll serve
```


### Deploy

```bash
bundle exec jekyll build
aws s3 sync --delete _site/ s3://www.stevenhalla.com
```
