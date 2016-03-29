# Pronto

[![Code Climate](https://codeclimate.com/github/mmozuras/pronto.png)](https://codeclimate.com/github/mmozuras/pronto)
[![Build Status](https://secure.travis-ci.org/mmozuras/pronto.png)](http://travis-ci.org/mmozuras/pronto)
[![Gem Version](https://badge.fury.io/rb/pronto.png)](http://badge.fury.io/rb/pronto)
[![Dependency Status](https://gemnasium.com/mmozuras/pronto.png)](https://gemnasium.com/mmozuras/pronto)
[![Inline docs](http://inch-ci.org/github/mmozuras/pronto.png)](http://inch-ci.org/github/mmozuras/pronto)

**Pronto** runs analysis quickly by checking only the relevant changes. Created to
be used on [pull requests](#github-integration), but also works [locally](#local-changes) and integrates with [GitLab](#gitlab-integration).
Perfect if want to find out quickly if branch introduces changes that conform
to your [styleguide](https://github.com/mmozuras/pronto-rubocop), [are DRY](https://github.com/mmozuras/pronto-flay), [don't introduce security holes](https://github.com/mmozuras/pronto-brakeman) and [more](#runners).

![Pronto demo](pronto.gif "")

* [Installation](#installation)
* [Usage](#usage)
    * [Local Changes](#local-changes)
    * [GitHub Integration](#github-integration)
    * [GitLab Integration](#gitlab-integration)
* [Configuration](#configuration)
* [Runners](#runners)
* [Articles](#articles)
* [Changelog](#changelog)
* [Copyright](#copyright)

## Installation

**Pronto**'s installation is standard for a Ruby gem:

```sh
$ gem install pronto
```

You'll also want to install some [runners](#runners) to go along with the main gem:

```sh
$ gem install pronto-rubocop
$ gem install pronto-flay
```

If you'd rather install Pronto using `bundler`, you don't need to require it,
unless you're gonna run it from Ruby (via Rake task, for example):

```ruby
gem 'pronto'
gem 'pronto-rubocop', require: false
gem 'pronto-flay', require: false
```

## Usage

Pronto runs the checks on a diff between the current HEAD and the provided commit-ish (default is master).

### Local Changes

Navigate to the repository you want to run Pronto on, and:

```sh
git checkout feature/branch

# Analyze diff of committed changes on current branch and master:
pronto run

# Analyze diff of uncommitted changes and master:
pronto run --index

# Analyze *all* changes since the *initial* commit (may take some time):
pronto run --commit=$(git log --pretty=format:%H | tail -1)
```

Just run `pronto` without any arguments to see what Pronto is capable of.

Available Options

Command flag     | Description
-----------------|------------------------------------------------------------
`--exit-code`    | Exits with non-zero code if there were any warnings/errors. Code will be the number of warnings/errors found.
`-c/--commit`    | Commit for the diff.
`-i/--index`     | Analyze changes in git index (staging area).
`-r/--runner`    | Run only the passed runners.
`-f/--formatters`| Pick output formatters.

### GitHub Integration

You can run Pronto as a step of your CI builds and get the results as comments
on GitHub commits using `GithubFormatter` or `GithubPullRequestFormatter`.

Add Pronto runners you want to use to your Gemfile:

Set the GITHUB_ACCESS_TOKEN environment variable or value in `.pronto.yml` to
[OAuth token](https://help.github.com/articles/creating-an-access-token-for-command-line-use) that has access to the repository.

Then just run it:

```sh
$ GITHUB_ACCESS_TOKEN=token pronto run -f github -c origin/master
```

or, if you want comments to appear on pull request diff, instead of commit:

```sh
$ GITHUB_ACCESS_TOKEN=token PULL_REQUEST_ID=id pronto run -f github_pr -c origin/master
```

As an alternative, you can also set up a rake task:

```ruby
Pronto::GemNames.new.to_a.each { |gem_name| require "pronto/#{gem_name}" }

formatter = Pronto::Formatter::GithubFormatter.new # or GithubPullRequestFormatter
Pronto.run('origin/master', '.', formatter)
```

### GitLab Integration

You can run Pronto as a step of your CI builds and get the results as comments
on GitLab commits using `GitlabFormatter`.

**note: this requires at least GitLab v7.5.0**

Set the `GITLAB_API_ENDPOINT` environment variable or value in `.pronto.yml` to
your API endpoint URL. If you are using Gitlab.com's hosted service your
endpoint will be `https://gitlab.com/api/v3`.
Set the `GITLAB_API_PRIVATE_TOKEN` environment variable or value in `.pronto.yml
to your Gitlab private token which you can find in your account settings.

Then just run it:

```sh
$ GITLAB_API_ENDPOINT="https://gitlab.com/api/v3" GITLAB_API_PRIVATE_TOKEN=token pronto run -f gitlab -c origin/master
```

As an alternative, you can also set up a rake task:

```ruby
Pronto::GemNames.new.to_a.each { |gem_name| require "pronto/#{gem_name}" }

formatter = Pronto::Formatter::GitlabFormatter.new
Pronto.run('origin/master', '.', formatter)
```

## Configuration

The behavior of Pronto can be controlled via the `.pronto.yml` configuration
file. It must be placed in your project directory.

The file has the following format:

```yaml
all:
  exclude:
    - 'spec/**/*'
github:
  slug: mmozuras/pronto
  access_token: B26354
  api_endpoint: https://api.github.com/
  web_endpoint: https://github.com/
gitlab:
  slug: mmozuras/pronto,
  api_private_token: 46751,
  api_endpoint: https://api.vinted.com/gitlab
max_warnings: 150
verbose: false
```

All properties that can be specified via `.pronto.yml`, can also be specified
via environment variables. Their names will be the upcased path to the property.
For example: `GITHUB_SLUG` or `GITLAB_API_PRIVATE_TOKEN`. Environment variables
will always take precedence over values in configuration file.

## Runners

Pronto can run various tools and libraries, as long as there's a runner for it.
Currently available:

* [pronto-brakeman](https://github.com/mmozuras/pronto-brakeman)
* [pronto-coffeelint](https://github.com/siebertm/pronto-coffeelint)
* [pronto-eslint](https://github.com/mmozuras/pronto-eslint)
* [pronto-fasterer](https://github.com/mmozuras/pronto-fasterer)
* [pronto-flay](https://github.com/mmozuras/pronto-flay)
* [pronto-foodcritic](https://github.com/mmozuras/pronto-foodcritic)
* [pronto-jscs](https://github.com/spajus/pronto-jscs)
* [pronto-jshint](https://github.com/mmozuras/pronto-jshint)
* [pronto-haml](https://github.com/mmozuras/pronto-haml)
* [pronto-poper](https://github.com/mmozuras/pronto-poper)
* [pronto-rails_best_practices](https://github.com/mmozuras/pronto-rails_best_practices)
* [pronto-rails_schema](https://github.com/raimondasv/pronto-rails_schema)
* [pronto-reek](https://github.com/mmozuras/pronto-reek)
* [pronto-rubocop](https://github.com/mmozuras/pronto-rubocop)
* [pronto-scss](https://github.com/mmozuras/pronto-scss)
* [pronto-spell](https://github.com/mmozuras/pronto-spell)
* [pronto-swiftlint](https://github.com/ajanauskas/pronto-swiftlint)
* [pronto-tailor](https://github.com/ajanauskas/pronto-tailor)

## Articles

Articles to help you to get started:

* [Automating code review with Pronto (and friends)](http://everydayrails.com/2015/02/17/pronto-ruby-code-review.html)
* [Setup Pronto with CircleCI](https://medium.com/@MaximAbramchuk/circleci-github-pr-commenting-ruby-scss-coffeescript-javascript-git-and-etc-fbcbe2a378a5#.gk5f14p3j)
* [Continuous Static Analysis using Pronto](http://codingfearlessly.com/2014/11/06/continuous-static-analysis/)
* [Pronto and git hooks](http://elliotthilaire.net/gem-pronto-and-git-hooks/)
* [How to end fruitless dev discussions about your project’s code style?](https://medium.com/appaloosa-store-engineering/how-to-end-fruitless-dev-discussions-about-your-project-s-code-style-245070bff6d4)

Make a Pull Request to add something you wrote or found useful.

## Changelog

**Pronto**'s changelog is available [here](CHANGELOG.md).

## Copyright

Copyright (c) 2013-2016 Mindaugas Mozūras. See [LICENSE](LICENSE) for further details.
