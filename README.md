logsearch-boshworkspace
=======================

The fastest way to deploy [Logsearch](http://www.logsearch.io) in combination with [Cloud Foundry](http://www.cloudfoundry.org) onto [bosh-lite](https://github.com/cloudfoundry/bosh-lite).

Preparation
-----------

To get started you will need a running bosh-lite. Get yours by following the instructions [here](https://github.com/cloudfoundry/bosh-lite#install-bosh-lite)

Next step is setting up this repository

```
git clone https://github.com/cloudfoundry-community/logsearch-boshworkspace.git
cd logsearch-boshworkspace
bundle install
```

Filters
-------

The logsearch community has created some pre-built filters. They are distributed as plugin repositories (such as https://github.com/logsearch/logsearch-filters-common).

A `rake` task is provided to fetch a subset of these plugin filters and update the `templates/` to include the logstash filters.

```
rake plugins:update
```

Currently some plugins test suites fail - you can continue with:

```
rake plugins:update_templates
```

Open issues for failing test suites for plugins:

-	[![cf](https://github-shields.cfapps.io/github/logsearch/logsearch-filters-cf/issues/11.svg)](https://github-shields.cfapps.io/github/logsearch/logsearch-filters-cf/issues/11)
-	[![website](https://github-shields.cfapps.io/github/logsearch/logsearch-for-websites/issues/3.svg)](https://github-shields.cfapps.io/github/logsearch/logsearch-for-websites/issues/3)

Deploy
------

This section includes instructions for configuration and deployment of Logsearch.

### Deploy Logsearch to bosh-lite

```
bosh deployment logsearch-warden
bosh prepare deployment
bosh deploy
```

### Deploy Logsearch to AWS VPC

Copy the `logsearch-aws-vpc.yml` or `logsearch-aws-vpc-cf-route.yml` to a new file that you will edit:

```
cp deployments/logsearch-aws-vpc-cf-route.yml \
   deployments/my-logsearch.yml
```

Edit `deployments/my-logsearch.yml` with your:

-	director UUID (`bosh status --uuid`\)

### Update Cloud Foundry with Syslog enabled

You can now re-deploy Cloud Foundry with `syslog` emitting to your Logsearch Add the following to your Cloud Foundry deployment manifest.

First, get the IP or hostname for `ingestor/0` VM:

```
$ bosh vms --dns
Deployment `logsearch-aws-test'

+------------+---------+---------------+------------+-------------------------------------------------+
| Job/index  | State   | Resource Pool | IPs        | DNS A records                                   |
+------------+---------+---------------+------------+------------+------------------------------------+
| ingestor/0 | running | common        | 10.10.6.7  | 0.ingestor.default.logsearch-aws-test.microbosh |
...
```

In the example above, use either `10.10.6.7` or `0.ingestor.default.logsearch-aws-test.microbosh`

Add the following to your Cloud Foundry deployment manifest (global properties as it is for all job templates):

```
properties:
  syslog_daemon_config:
    address: 10.10.6.7
    port: 5515
    transport: relp
```

Now redeploy Cloud Foundry:

```
$ bosh deploy
```

### Access Kibana UI via Cloud Foundry router

Instead of `logsearch-warden`, use `logsearch-warden-cf-route` to make Kibana UI accessible via the public CF router (`logs.DOMAIN`\)

### Play time

Now you can browse to the Kibana dashboard [here](http://10.244.2.2/_plugin/kibana/index.html#/dashboard/file/logstash.json)
