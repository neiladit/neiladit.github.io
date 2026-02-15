# Quick build and run


(assumes prerequisites already installed)

- Ensure the repo uses the correct Ruby for this project (recommended: `3.1.4`):

```bash
# set the repo-local Ruby (rbenv)
rbenv local 3.1.4
ruby -v   # should show 3.1.4
```

- Install Bundler and gems, then build and serve:

```bash
gem install bundler
bundle install
bundle exec jekyll build
bundle exec jekyll serve --livereload --host 127.0.0.1
```

## Short instructions to build and serve this Jekyll site on macOS

Prerequisites
- macOS with Terminal (bash). Homebrew recommended.
- Recommended Ruby management: `rbenv` (or `asdf`). Do NOT remove system Ruby.

Quick start (recommended)
1. Install Homebrew (if you don't have it):

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

2. Install rbenv + ruby-build and initialize for this shell:

```bash
brew install rbenv ruby-build
echo 'export PATH="$(brew --prefix rbenv)/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(rbenv init - bash)"' >> ~/.bash_profile
source ~/.bash_profile
```

3. Install a supported Ruby and set it locally for the repo. `github-pages` in this repo requires Ruby < 4.0; use 3.1.4 for compatibility:

```bash
rbenv install -s 3.1.4
rbenv local 3.1.4
ruby -v   # should show 3.1.4
```

4. Install Bundler and the gems from the `Gemfile`:

```bash
gem install bundler
bundle install
```

Notes about Bundler / Gemfile.lock
- If `bundle` warns your lockfile was generated with a different Bundler, prefer updating the lockfile with your Bundler version:

```bash
# check bundler version
bundle -v
# update lockfile to current bundler
bundle _<your-version>_ update --bundler
bundle install
```



