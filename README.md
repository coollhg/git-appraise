# Distributed Code Review For Git
[![Build Status](https://travis-ci.org/google/git-appraise.svg?branch=master)](https://travis-ci.org/google/git-appraise)

This repo contains a command line tool for performing code reviews on git
repositories.

## Overview

This tool is a *distributed* code review system for git repos.

By "distributed", we mean that code reviews are stored inside of the repository
as git objects. Every developer on your team has their own copy of the review
history that they can push or pull. When pulling, updates from the remote
repo are automatically merged by the tool.

This design removes the need for any sort of server-side setup. As a result,
this tool can work with any git hosting provider, and the only setup required
is installing the client on your workstation.

Additionally, code reviews can be conducted across multiple hosting providers.
The list of forks of a repository is also stored in the repository as git
objects, allowing code reviews to be pulled from every registered fork.

## Installation

Assuming you have the [Go tools installed](https://golang.org/doc/install), run
the following command:

    go get github.com/google/git-appraise/git-appraise

Then, either make sure that `${GOPATH}/bin` is in your PATH, or explicitly add the
"appraise" git alias by running the following command.

    git config --global alias.appraise '!'"${GOPATH}/bin/git-appraise"

#### Windows:

    git config --global alias.appraise "!%GOPATH%/bin/git-appraise.exe"

## Requirements

This tool expects to run in an environment with the following attributes:

1.  The git command line tool is installed, and included in the PATH.
2.  The tool is run from within a git repo.
3.  The git command line tool is configured with the credentials it needs to
    push to and pull from the remote repos.

## Usage

Requesting a code review:

    git appraise request

Pushing code reviews to a remote:

    git appraise push [<remote>]

Pulling code reviews from a remote:

    git appraise pull [<remote>]

Listing open code reviews:

    git appraise list

Showing the status of the current review, including comments:

    git appraise show

Showing the diff of a review:

    git appraise show --diff [--diff-opts "<diff-options>"] [<review-hash>]

Commenting on a review:

    git appraise comment -m "<message>" [-f <file> [-l <line>]] [<review-hash>]

Accepting the changes in a review:

    git appraise accept [-m "<message>"] [<review-hash>]

Submitting the current review:

    git appraise submit [--merge | --rebase]

Adding a fork:

    git appraise fork add [--owner "<contributor-email>"]+ <name> <url>

A more detailed getting started doc is available [here](docs/tutorial.md).

Instructions for using `git-appraise` with multiple forks can be found
[here](docs/forks.md).

## Metadata

The code review data is stored in [git-notes](https://git-scm.com/docs/git-notes),
using the formats described below. Each item stored is written as a single
line of JSON, and is written with at most one such item per line. This allows
the git notes to be automatically merged using the "cat\_sort\_uniq" strategy.

Since these notes are not in a human-friendly form, all of the refs used to
track them start with the prefix "refs/notes/devtools". This helps make it
clear that these are meant to be read and written by automated tools.

When a field named "v" appears in one of these notes, it is used to denote
the version of the metadata format being used. If that field is missing, then
it defaults to the value 0, which corresponds to this initial version of the
formats.

### Code Review Requests

Code review requests are stored in the "refs/notes/devtools/reviews" ref, and
annotate the first revision in a review. They must conform to the
[request schema](schema/request.json).

If there are multiple requests for a single commit, then they are sorted by
timestamp and the final request is treated as the current one. This sorting
should be done in a stable manner, so that if there are multiple requests
with the same timestamp, then the last such request in the note is treated
as the current one.

This design allows a user to update a review request by re-running the
`git appraise request` command.

### Continuous Integration Status

Continuous integration build and test results are stored in the
"refs/notes/devtools/ci" ref, and annotate the revision that was built and
tested. They must conform to the [ci schema](schema/ci.json).

### Robot Comments

Robot comments are comments generated by static analysis tools. These are
stored in the "refs/notes/devtools/analyses" ref, and annotate the revision.
They must conform to the [analysis schema](schema/analysis.json).

### Review Comments

Review comments are comments that were written by a person rather than by a
machine. These are stored in the "refs/notes/devtools/discuss" ref, and
annotate the first revision in the review. They must conform to the
[comment schema](schema/comment.json).

## Integrations

### Libraries

  - [Go (use git-appraise itself)](https://github.com/google/git-appraise/blob/master/review/review.go)
  - [Rust](https://github.com/Nemo157/git-appraise-rs)

### Graphical User Interfaces

  - [Git-Appraise-Web](https://github.com/google/git-appraise-web)

### Plugins

  - [Eclipse](https://github.com/google/git-appraise-eclipse)
  - [Jenkins](https://github.com/jenkinsci/google-git-notes-publisher-plugin)

### Mirrors to other systems

  - [GitHub Pull Requests](https://github.com/google/git-pull-request-mirror)
  - [Phabricator Revisions](https://github.com/google/git-phabricator-mirror)

## Contributing

Please see [the CONTRIBUTING file](CONTRIBUTING.md) for information on contributing to Git Appraise.
