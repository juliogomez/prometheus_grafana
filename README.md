# What is Prometheus?

[Prometheus](http://prometheus.io) is an open-source systems monitoring and alerting toolkit.

Its main features are:

* a multi-dimensional data model with time series data identified by metric name and key/value pairs
* a flexible query language to leverage this dimensionality
* no reliance on distributed storage; single server nodes are autonomous
* time series collection happens via a pull model over HTTP
* pushing time series is supported via an intermediary gateway
* targets are discovered via service discovery or static configuration
* multiple modes of graphing and dashboarding support

Prometheus scrapes metrics from instrumented jobs. It stores all scraped samples locally and runs rules over this data to either aggregate and record new time series from existing data or generate alerts. Grafana, or other API consumers, can be used to visualize the collected data.

Prometheus works well for recording any purely numeric time series. It fits both machine-centric monitoring as well as monitoring of highly dynamic service-oriented architectures. In a world of microservices, its support for multi-dimensional data collection and querying is a particular strength.

# What is Grafana

[Grafana](https://grafana.com) is an open source metric analytics & visualization suite. It is most commonly used for visualizing time series data for infrastructure and application analytics but many use it in other domains including industrial sensors, home automation, weather, and process control.

Grafana supports many different storage backends for your time series data (Data Source), including of course Prometheus. Each Data Source has a specific Query Editor that is customized for the features and capabilities that the particular Data Source exposes.

# Our demo

We will deploy a number of exporters for Prometheus to scrape from, and then use Grafana to show some nice graphs based on the info offered by Prometheus.

To make things easier, and not have to deploy our own infrastructure, we will use [play-with-docker](https://labs.play-with-docker.com) (PWD) to create a Swarm cluster.

## Create your Swarm cluster

The first thing we need to do is to create a Swarm cluster where we can deploy all our containers for this demo. The easiest way to do it is to go to [play-with-docker.com](https://labs.play-with-docker.com) (PWD).

Click on 'Login' and use your Docker username/password (they are completely free if you do not have them, so please go ahead and register if required). Then click on 'Start' and you have 4 hours ahead of Docker resources to play with. You could now create individual hosts and configure them as managers & workers for your Swarm, but let's make it easier. Click on the *wrench* icon on the left part of the screen and select the template '3 managers and 2 workers'. This will automatically create a Swarm cluster for you, ready to use!

## Clone the code repo

PWD instances include a git client, so you can download code from github repos. Please go to 'manager1' and clone the following repo:

```
git clone https://github.com/juliogomez/prometheus_grafana.git
```

Then go into the newly created directory:

```
cd prometheus_grafana
```

If you run `ls` you will see there are a couple of [YAML](https://en.wikipedia.org/wiki/YAML) files in there.

* 'docker-compose.yml' includes all different services we will be running and their associated configs.
* 'prometheus.yml' defines how Prometheus will scrape those different microservices

This repo uses tags, so you can easily switch between different versions of these files. You can learn more on how to use git tags [here](https://github.com/juliogomez/git_tags).
