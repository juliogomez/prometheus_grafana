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

# What is Grafana?

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

## Deploy Prometheus

Let's switch to our first version of the code with:

```
git checkout 1
```

If you `cat docker-compose.yml` you will see the file creates a new overlay network for all containers to use, and use 'prometheus.yml' as a configuration file. Then it runs a container with the prometheus image, serving its content at port 9090.

Please `cat prometheus.yml` to see the scraping time intervals and the exporter to scrape. In this initial step we will scrape Prometheus itself for its own metrics. As you will probably notice it does not need to define Prometheus' IP address, but rather just its name. This is because of the service discovery capabilities included in Prometheus.

0k, it is now time to see it working. Let's deploy it in our Swarm cluster by running the following command:

```
docker stack deploy -c docker-compose.yml my-stack
```

Use `docker stack ls` to verify a new stack has been created, and its only service (Prometheus itself).

You can see running services with:

```
docker service ls
```

You can see if the service is running correctly, and also in what specific node of your Swarm cluster the specific service task has been deployed with:

```
docker stack ps my-stack
```

You may have noticed there is now a new link on top of your PWD screen, marked with **9090**. Click and you will get Prometheus web interface. From there let's check several things:

* Go to 'Status' - 'Configuration' and check that Prometheus has the correct config as defined in your 'prometheus.yml' file.
* Go to 'Status' - 'Targets' and verify that our scraping target (prometheus itself in this initial step) appears as UP.

Now click on 'Graph' and from the drop-down '- insert metric at cursor -' menu choose one of the metrics (for example 'prometheus_evaluator_duration_seconds'), click on 'Execute' and then go to 'Graph'. You will start seeing some basic graphs for this metric. Prometheus include a very basic solution for graphs, so please note they are not *dynamic / live*, meaning they are not automatically updated, you need to click on 'Execute' to get them updated. That is exactly why we need a good solution for graphs, like Grafana, that we will deploy later.

## Deploy Container Advisor

[cAdvisor](https://github.com/google/cadvisor) provides you with an understanding of the resource usage and performance characteristics of running containers.

Let's switch to v2 of our files with:

```
git tag 2
```

If you examine again the two files in your directory, you will see they both include now the required config to run cAdvisor. One difference you will notice is that we will deploy cAdvisor in a *global* mode. This means we will have one instance of this service running in every node.

Let's stop the old stack and deploy a new one with:

```
docker stack rm my-stack
docker stack deploy -c docker-compose.yml my-stack
```

The same way you did in the previous step, please go to Prometheus web interface (**9090** link on top of the PWD screen), check that the configuration looks correct, and the targets are all 'UP' (1x Prometheus and 5x cAdvisor instances). In the 'Graph' section you will find more available metrics in the drop-down menu, specifically the ones starting with 'container_'.

You may choose on of these container metrics (for example 'container_cpu_usage_seconds_total'), click on 'Execute' and go to 'Graph'. Again you will see a basic graph for some container metrics.

## Deploy Node Exporter

[Node Exporter](https://github.com/prometheus/node_exporter) will export hardware and OS metrics for each one of the Swarm nodes.

Let's switch to v3 of our files with:

```
git tag 3
```

Now both files in your directory will include the required config to run Node Exporter in every Swarm node.

Let's stop the old stack and deploy a new one with:

```
docker stack rm my-stack
docker stack deploy -c docker-compose.yml my-stack
```

The same way you did in the previous step, please go to Prometheus web interface (**9090** link on top of the PWD screen), check that the configuration looks correct, and the targets are all 'UP' (1x Prometheus, 5x cAdvisor, 5x node-exporter instances). In the 'Graph' section you will find more available metrics in the drop-down menu, specifically the ones starting with 'node_'.

You may choose on of these container metrics (for example 'node_memory_Active'), click on 'Execute' and go to 'Graph'. Again you will see a basic graph for some node metrics.

## Deploy Postgres, Traefik and Grafana

Using the same procedure let's now deploy a couple of additional exporters.

* [Postgres](https://hub.docker.com/_/postgres/) (or PostgreSQL) is an object-relational database management system, and it has its own [exporter](https://github.com/wrouesnel/postgres_exporter).

* [Traefik](https://traefik.io/) is a modern HTTP reverse proxy and load balancer made to deploy microservices with ease.

Let's switch to v4 of our files with:

```
git tag 4
```

Now both files in your directory will include the required config to run Postgres, Traefik and Grafana services.

Let's stop the old stack and deploy a new one with:

```
docker stack rm my-stack
docker stack deploy -c docker-compose.yml my-stack
```

The same way you did in the previous step, please go to Prometheus web interface (**9090** link on top of the PWD screen), check that the configuration looks correct, and the targets are all 'UP' (1x Prometheus, 5x cAdvisor, 5x node-exporter, 1x Postgres, 3x Traefik instances). In the 'Graph' section you will find more available metrics in the drop-down menu.

Additionally you can now check Grafana web interface, available via the new **3000** link in the top part of the PWD screen. Click on it and login with admin/admin. Then click on the icon situated on the top left and choose 'Dashboards' - 'Import'. Under 'Grafana.com Dashboard' type the number of one of the available templates: for example 609, and 'Load'. Under 'Options' click on 'Select a Prometheus data source' and choose 'Prometheus'. For both 'ElasticSearch' drop-down menus select 'Alerts'. Click on 'Import' and you will start seeing your new dashboard. On the top right click on 'Last 24 hours' and choose 'Last 15 minutes'.

**Congrats!!!** Now you have a beautiful Grafana dashboard that dynamically shows useful live information on multiple container and node metrics!

Bonus points: you may import another example dashboard template, for example number 893, by following the same procedure.
