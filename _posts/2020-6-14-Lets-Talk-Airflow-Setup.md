---
layout: post
title: Let's Talk Airflow Setup
---

Approaching Airflow can seem a little daunting. There are a few hundred Medium articles out there telling you how to set up Airflow, write a DAG, test a "Hello, World!" ETL. Usually the there is a lot to set up before any development, experimentation, or learning can take place. While the articles are helpful, they are often just way too long.

**Pre-built Docker images will save us time and let us start experimenting with Airflow quickly.**

## Firing up an Airflow image
Make sure you have Docker installed.

If you want to develop dags in your local IDE, I recommend using the docker-compose file provided in the [Puckel](https://github.com/puckel/docker-airflow) repository. This will mount the `dags/` folder as a volume and allow you to edit dags on the fly and see your changes updated in the UI. This workflow is my preferred way to develop.  

{% highlight shell %}
 git clone https://github.com/puckel/docker-airflow.git && cd docker-airflow
 docker-compose -f docker-compose-LocalExecutor.yml up -d
{% endhighlight %}

navigate to localhost:8080 in your web browser to check out the UI.

You can shut down your instance from the same shell with `docker-compose down`

## Some Details
You'd be surprised to find out that Apache doesn't really have a "production ready" image yet. Most of their images require some tinkering to get up and running. The reason the Puckel image has gained so much traction is how easily you can fire up an Airflow instance. [Here's a link](https://cwiki.apache.org/confluence/display/AIRFLOW/AIP-26+Production-ready+Airflow+Docker+Image+and+helm+chart) to the Airflow Jira request for a production-ready image.

It is also worth looking at the [GitHub issue page](https://github.com/apache/airflow/issues/8605) since Puckel's repo hasn't pushed a release since February of 2020. There are also some great ideas for docker-compose files in that thread if you want to dig into the details.
