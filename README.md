# Personal blog

Initial setup, for stock macOS 15.x:

```sh
$ sudo gem install bundler -v 2.4.22
$ sudo bundle update --bundler
```

Run locally with drafts on:

```sh
$ rake drafts
```

Check deploy locally (e.g linting):

```sh
$ tools/deploy.sh --dry-run
```
