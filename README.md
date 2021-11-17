# OpenShift Streams Datadog OpenMetrics Integration

This guide explains how to deploy a Prometheus instance on OpenShift 4.x that
will capture metrics from OpenShift Streams for Apache Kafka. A Datadog Agent
is then used to forward the captured metrics to Datadog for further analysis.

_NOTE: This setup uses a close to default **datadog-values.yaml** file. The Kube TLS verify is disabled, and Prometheus scraping is enabled. Review and edit this file to suit your own needs._

![](/architecture.png)

## Pre-requisites

* [OpenShift Streams for Apache Kafka instance ID and Service Account](https://console.redhat.com/application-services/streams/kafkas)
* [OpenShift 4.x cluster](https://console.redhat.com/openshift/create)
* [OpenShift 4.x CLI (`oc`)](https://console.redhat.com/openshift/downloads)
* [Helm CLI 3.7+ (`helm`)](https://helm.sh/docs/intro/install/)
* Datadog Account
* Datadog API Key

## Preparation

You'll need to install the Prometheus Operator and edit the
*kafka-fedetate.yaml* to contain valid credentials that are used to scrape
Kafka instance metrics.

### Install the Prometheus Operator
1. Login to your OpenShift cluster console.
1. Create a new Project, or select an existing one.
1. Select OperatorHub from the side-menu.
1. Search for the "Prometheus Operator" and install it.
![](/images/operator-install.png)

## Prepare the Scrape Config

1. Create a copy of the *kafka-federate.yaml.template* file named *kafka-federate.yaml*.
1. Edit the `metrics_path` URL in the *kafka-federate.yaml* file to contain the ID of your OpenShift Streams for Apache Kafka instance.
![](/images/kafka-instance-id.png)
1. Edit the `client_id` and `client_secret` in the *kafka-federate.yaml* file to contain valid Service Account credentials for your Kafka instance.

## Setup

1. Login to your OpenShift cluster console.
1. Click the dropdown in the top-right of the OpenShift cluster console and use the **Copy login command** link.
1. Paste the login command into a terminal session.
1. Select your project/namespace:
    ```bash
    oc project <your-project-name>
    ```
1. Run the following command to create a Secret containing the Kafka scrape config:
    ```bash
    oc create secret generic additional-scrape-configs --from-file=kafka-federate.yaml --dry-run -o yaml | oc create -f -
    ```
1. Next, create a Prometheus instance annotated for Datadog scraping:
    ```bash
    oc create -f prometheus.yaml
    ```
1. Deploy the Datadog Agent:
    ```bash
    export DD_API_KEY="your api key goes here"
    export DD_SITE="datadoghq.com"

    helm repo add datadog https://helm.datadoghq.com

    helm install datadog-agent datadog/datadog \
    -f datadog-values.yaml \
    --set datadog.site=$DD_SITE \
    --set datadog.apiKey=$DD_API_KEY
    ```

At this point the Datadog agent is running on your cluster nodes, scraping Prometheus, and reporting to Datadog.

![](/images/running-pods.png)

You can now create Datadog Dashboards using the exported metrics.

