# Helder

**Daniel Colon - 20/08/2022**

This is my homepage built using HUGO.
[Hosted at helder.uk](https://helder.uk)

## Theme

The page uses the
[papermod theme](https://github.com/adityatelange/hugo-PaperMod)

The theme is cloned in as a submodule. When pulling the repository make sure to
run:

`git submodule update --init --recursive`

## Deployment

This page is hosted using an S3 bucket serving cloudfront. Whilst security
though obscurity is bad practice (and I don't rely on it!) the terraform
scripts used to set this all up are not public, although I may write an article
on it at some point. In summary:
I wrote a static site terraform module, this module does a few things:

- It creates an AWS organisation with separate test and production accounts
  with appropriate permission boundaries set. These are used by terraform to
  create the static site deployments.
- Using the accounts in these organisations, it creates a test and production s3
  bucket for the website, and sets up a Cloudfront distribution for these
  buckets. This includes a few things like an allowlist for certain countries,
  SSL certificates, and using cloudfront functions to rewrite to friendly URLs.
- It creates an IAM user with strict boundaries that can manage the files in the
  relevant S3 buckets.
- It creates this github repository and sets the IAM user's credentials as
  secrets.

Once this is in place, the workflows in this repository can use these
credentials to publish the website to the S3 bucket whenever a push is made to
the `test` or `production` branches. No-one but me has permissions on github
to do this.

Development takes place on the `master` branch, this is then merged into the
`test` and finally the `production` branch for release.