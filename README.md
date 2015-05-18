# maek-deploy
Extension for rack apps: Add the gem, change the config file, "rake deploy {production|staging}".

Overall aim: single command deployment from nothing.

# Raison d'etre
Provisioning servers is hard. Deploying rails apps, with versioning, is hard. Scaling rails apps is hard. Providing secrets to those rails apps is hard.

There are services which abstract this away for you, but they're expensive, and some of them (here's looking at you, Elastic Beanstalk) are opaque and difficult to understand.

There are services you can add and configure to manage deployment, but these are "catch-all" solutions which are entire platforms in and of themselves. I just want to deploy a rails app and make it work on an IP.

## Proposed usage

Add to your gemfile, and `bundle install`:

    gem "maek-deploy", github: "kalleth/maek-deploy"

Run the generator:

    rails g maek-deploy install --environments production staging

This creates a directory for each environment the app will deploy to. The directories contain YAML files.

Edit the configuration files in `config/maek-deploy/$ENVIRONMENT`.

1. `config.yml`: hosting platform to use, basic configuration for how to deploy your app, which "extensions/features" to enable (i.e. delayed_job, etc)
2. `secrets.yml`: encrypted secrets which will be provided as environment variables to your app. Do not check this file in, instead check in the crypted version `secrets.yml.crypt-store` -- whenever it changes, on deploy you will be prompted for the decryption password.

Deploy your app:

    md deploy

## Under the hood

* On first run, creates an environment using the DigitalOcean API if it doesn't exist (i.e. boots servers, then provisions them with a chef/ansible/$whatever recipe).
* Wraps a capistrano configuration somehow to manage deployments, doesn't rebuild app
* allows `md scale` (similar to eb scale) to add or remove instances from the load balancer pool
* as soon as more than 1 instance is asked for it needs to set a "master" with a round-robin nginx instance and then alter the balancer config as instances are added and removed
* needs to handle deployments to multiple host platforms in a manner like "deploy, restart, test, re-add to the balancer pool"

## Stuff It Won't Do

I don't want this to be a catch-all solution; I want this to handle the 90% of rails website deployments abstracting away the platform to a couple of config file lines. 

Standard stuff like `delayed_job` and `sidekiq` and perhaps `clockwork` should be accounted for -- maybe as a background worker -- but I am *not* looking to allow people to reimplement chef in our yaml files.
