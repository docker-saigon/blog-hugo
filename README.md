# Docker Saigon blog - source

content for the [docker-saigon.github.io](http://docker-saigon.github.io) blog, powered by:

![Hugo](https://raw.githubusercontent.com/spf13/hugo/master/docs/static/img/hugo-logo.png) (v0.14)

HUGO IS AMAZING!

## Overview

Hugo is a static site generator written in [Go]().
It is optimized for speed, easy use and configurability.
Hugo takes a directory with content and templates and renders them into a full HTML website.

Hugo relies on Markdown files with front matter for meta data.
And you can run Hugo from any directory.

### Theme

Currently we are using the [startbootstrap-clean-blog](https://github.com/IronSummitMedia/startbootstrap-clean-blog) Theme.
The theme has been added as a `git submodule`. 

If you want to work on the blog, you will need to use the `--recursive` option
when you `git clone` this repository.

## Writing a blog post

To add a blog posts to this blog, follow these steps:

1. Fork this repository and its submodules `git clone --recursive git@github.com:docker-saigon/blog-hugo`

1. Get the [Hugo binary](https://github.com/spf13/hugo/releases/latest) for your platform & put it on `PATH` as `hugo`

1. Ensure everything is working by using `hugo server -w -d dev/` to run a live reload server accessible on [http://localhost:1313](http://localhost:1313)

1. Add a new post via cli, using `hugo new post/<title>` & finalize the post using the live reload server. [Reference](https://gohugo.io/commands/hugo_new/)

1. Open a pull-request on github for peer review of your new post.

## Publishing updates to 

1. Pull Updates from origin

1. If `public/` folder does not exist yet, clone current site to `public/` folder: `git submodule add git@github.com:docker-saigon/docker-saigon.github.io public/`

1. Run `hugo` to generate the new site with new content overwriting `public/`

1. Use `./publish.sh [commit message]` script to push new static site to the `docker-saigon/docker-saigon.github.io` repository on github

`./publish.sh` will move into the `public/` folder, commit all the changes and push them to the website.
