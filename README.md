# Web
[![pages-build-deployment](https://github.com/cgutierr-zgz/cgutierr-zgz.github.io/actions/workflows/pages/pages-build-deployment/badge.svg)](https://github.com/cgutierr-zgz/cgutierr-zgz.github.io/actions/workflows/pages/pages-build-deployment)

Built with hugo

---

## Adding a new post

```sh
hugo new --kind post posts/my_post.md
```

For emojis please refer to: [gitmoji](https://gitmoji.dev), or this [cheatsheet](https://www.webfx.com/tools/emoji-cheat-sheet)
For thumbnails please refer to: [Snappa](https://snappa.com)

## Start Hugo server

```sh
hugo server -D --noHTTPCache
```

## Deploying

Run `hugo` and use the `public/` directory to upload to the production server

## Theme

Making use of [congo](https://jpanther.github.io/congo/)
[Install the theme](https://jpanther.github.io/congo/docs/installation/#install-using-hugo)

1. Install go https://go.dev/dl/
2. Install hugo https://gohugo.io/getting-started/installing/
3. Install the theme

```sh
# If you're managing your project on GitHub
hugo mod init github.com/<username>/<repo-name>

# If you're managing your project locally
hugo mod init my-project
```

To [update](https://jpanther.github.io/congo/docs/installation/#update-using-hugo) the theme run:

```sh
hugo mod get -u

# You can also run this to delete cache
hugo mod clean
```

## Favicon

Favicon(s) can be generated by [Favicon.io](Favicon.io) and can be simply put in /static folder.
