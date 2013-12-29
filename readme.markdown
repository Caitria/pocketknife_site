# Pocketknife Site

Here be the makeup for [`pocketknife`'s digital shingle](http://pocketknife.io).

It is a static website built using [jekyll](http://jekyllrb.com/).


## Installation

The only requirement is ruby. To install the required gems, run

```bash
# if you don't have bundler, install it with
[sudo] gem install bundler

# now use that to pull down the rest of the gems with
bundle install
```

That is all.


## Local

```bash
# run to compile, watch for changes, serve, and make you a sandwich
bundle exec jekyll serve --watch
```


## Pushing to S3

Perhaps not the best decision, but `bundle install` will install the
`s3_website` gem. This can be used to push the website to S3, given
that you have the correct s3_website.yml file.

To push:

```bash
# run this from the repo base directory
bundle exec s3_website push
```