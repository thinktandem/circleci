CircleCI
--------

This is a proof of concept repo that shows how to install [Lando](https://docs.devwithlando.io) on CircleCI and use it 
to perform your test suite.

Usage
-----

The main piece of value here is the [`.circlci/config.yml`](https://github.com/thinktandem/circleci/blob/master/.circleci/config.yml)
file.  You can either just take this file as your jumping off point and add it to your repo or fork this repo as a 
starting point for a new project.

Here is the full `.circleci/config.yml` file:

```yml
# https://circleci.com/docs/2.0/workflows/#using-workspaces-to-share-data-among-jobs
defaults: &defaults
  environment:
    TZ: "/usr/share/zoneinfo/America/New_York"
    TERM: dumb
version: 2
jobs:
  test:
    machine:
      image: circleci/classic:201711-01
    steps:
      - checkout
      - restore_cache:
          key: composer-{{ checksum "composer.lock" }}
      - run:
          name: Startup checks
          command: |
            sudo docker --version
            sudo docker-compose --version
            lsb_release -a
            export -p
      - run:
          name: Get Lando
          command: |
            sudo apt-get update
            sudo apt-get install cgroup-bin -y
            if [ ! -f /tmp/lando-latest.deb ]; then
              curl -fsSL -o /tmp/lando-latest.deb http://installer.kalabox.io/lando-latest-dev.deb
            fi
            sudo dpkg -i /tmp/lando-latest.deb
            lando version
      - run:
          name: Start Lando
          command: |
            lando start
      - save_cache:
          key: composer-{{ checksum "composer.lock" }}
          paths:
            - "vendor"
            - /tmp/lando-latest.deb
      - run:
          name: Test Lando
          command: |
            lando composer test

workflows:
  version: 2
  deployment:
    jobs:
      - test
```

PRs Will Trigger CircleCI
-------------------------

<img src="https://github.com/thinktandem/circleci/blob/master/circle-test.png" alt="CircleCI Tests Screenshot" align="center" width="644px;" />
