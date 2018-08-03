# trigger-travis.sh

This script triggers a new [Travis-CI](https://travis-ci.org/) job.
Ordinarily, a new Travis job is triggered when a commit is pushed to a
GitHub repository.  The `trigger-travis.sh` script provides a programmatic
way to trigger a new Travis job.

## Usage

```
trigger-travis.sh [--pro] [--branch BRANCH] GITHUBID GITHUBPROJECT TRAVIS_ACCESS_TOKEN [MESSAGE]
```

For example:
```
trigger-travis.sh typetools checker-framework `cat ~/private/.travis-access-token` "Trigger for testing"
```

`--pro` means to use `travis-ci.com` instead of `travis-ci.org`

`--branch BRANCH` means to use BRANCH instead of master.

`TRAVIS_ACCESS_TOKEN` is, or (in the example) `~/private/.travis-access-token` contains,
the Travis access token (see below for details).

`MESSAGE` is a string that will be displayed by Travis's web interface.
(For a commit push, Travis uses the commit message.)

## Travis access token

Your Travis access token is the text after "Your access token is " in
the output of this compound command:
```
  travis login && travis token
```

If the `travis` program isn't installed, then install it using either of these two
commands (whichever one works):
```
   gem install travis
   sudo apt-get install ruby-dev && sudo gem install travis
```
*Don't* do `sudo apt-get install travis` which installs a trajectory analyzer.

Note that the Travis access token output by `travis token` differs from the
Travis token available at https://travis-ci.org/profile .
If you store it in in a file, make sure the file is not readable by others,
for example by running:  chmod og-rwx ~/private/.travis-access-token

## Use in `.travis.yml`

To make one Travis build (if successful) trigger a different Travis build, do two things:

1. Set an environment variable `TRAVIS_ACCESS_TOKEN` by navigating to
  https://travis-ci.org/MYGITHUBID/MYGITHUBPROJECT/settings .
The `TRAVIS_ACCESS_TOKEN` environment variable will be set when Travis runs
the job, but it won't be visible to anyone browsing https://travis-ci.org/ .

2. Add the following to your `.travis.yml` file, where you replace
OTHERGITHUB* by a specific downstream project, but you leave
`$TRAVIS_ACCESS_TOKEN` as literal text:

```
jobs:
  include:
    - stage: trigger downstream
      jdk: oraclejdk8
      script: |
        echo "TRAVIS_BRANCH=$TRAVIS_BRANCH TRAVIS_PULL_REQUEST=$TRAVIS_PULL_REQUEST"
        if [[ ($TRAVIS_BRANCH == master) &&
              ($TRAVIS_PULL_REQUEST == false) ]] ; then
          curl -LO --retry 3 https://raw.github.com/mernst/plume-lib/master/bin/trigger-travis.sh
          sh trigger-travis.sh OTHERGITHUBID OTHERGITHUBPROJECT $TRAVIS_ACCESS_TOKEN
        fi
```

You don't need to supply a MESSAGE argument to `trigger-travis.sh`; it will
default to the current (upstream) repository, commit id, and one line of
the commit message.


## Credits and alternatives

Parts of this script were originally taken from
http://docs.travis-ci.com/user/triggering-builds/ .

An alternative to this script would be to install the Travis command-line
client and then run:
```
travis restart -r OTHERGITHUBID/OTHERGITHUBPROJECT
```

However, using `travis restart` is undesirable because it restarts an old
job, destroying its history.  This script starts a new job.
