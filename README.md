# Docker Saigon blog - source

this blog is powered by:

![Hugo](https://raw.githubusercontent.com/spf13/hugo/master/docs/static/img/hugo-logo.png) (v0.14)

## Overview

Hugo is a static site generator written in [Go][].
It is optimized for speed, easy use and configurability.
Hugo takes a directory with content and templates and renders them into a full HTML website.

Hugo relies on Markdown files with front matter for meta data.
And you can run Hugo from any directory.

### Theme

Currently we are using the [starbootstarp-clean-blog](https://github.com/IronSummitMedia/startbootstrap-clean-blog) Theme which has been added as a `git
submodule`. If you want to work on the blog, you will need to use the
`--recursive` option when you `git clone` this repository.

## Writing a blog post

To add a blog posts to this blog, follow these steps:

1. Fork this repository and its submodules `git clone --recursive git@github.com:docker-saigon/blog-hugo`

1. Get the [Hugo binary](https://github.com/spf13/hugo/releases/latest) for your platform & put it on `PATH` as `hugo`

1. Ensure everything is working by using `hugo server -w` to run a live reload server accessible on http://localhost:1313

1. Add a new post using `hugo new post/<title>` & finalize the post using the live reload server. [Reference](https://gohugo.io/commands/hugo_new/)

1. Open a pull-request on github for peer review of your new post.

## Admin publishing post

1. Pull latest changes

1. Run `hugo` to generate the new site in `public/`

1. Use `./publish.sh` script to push new static site to the `docker-saigon/blog-hugo` repository on github


