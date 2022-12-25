# Tale

## Features
- Easy installation
- Compatible with GitHub Pages
- Responsive design (looks just as good on mobile)
- Syntax highlighting, with the help of Pygments
- Markdown and HTML text formatting
- Pagination of posts
- Sticky posts
- Tags
- Excerpt management
- [Disqus comments (can be enabled if needed)](#enabling-comments)

## Installation

### Fork method
1. Fork this repository

2. Delete the unnecessary files/folders: `CODE_OF_CONDUCT.md`, `LICENSE`, `README.md`, `tale.gemspec`

3. Delete the `baseurl` line in `_config.yml`:

```yaml
baseurl:  "/tale"   # delete this line
```

## Usage
Once you've installed the theme, you're ready to work on your Jekyll site. To start off, I would recommend updating `_config.yml` with your site's details.

To build and serve your site, run:

```bash
$ bundle exec jekyll serve
```

And you're all set! Head over to http://127.0.0.1:4000/ to see your site in action.

### Enabling Comments
Comments are disabled by default. To enable them, look for the following line in `_config.yml` and change `jekyll-tale` to your site's Disqus id.

```yml
disqus: jekyll-tale
```

Next, add `comments: true` to the YAML front matter of the posts which you would like to enable comments for.