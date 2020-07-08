---
layout: post
title: Let's Talk dbt
---

[dbt](https://github.com/fishtown-analytics/dbt) (data build tool) is used to configure, structure, and visualize datbase objects including tables, views, and processes (the T part of ELT). Dbt lends itself to an event-driven architecture where we have some number of `raw` tables that are routinely populated in the databse. Any tables that are downstream of these raw tables are materialized with dbt. It is an important distinction here that dbt will not help you with getting raw data into your database, but will help you model and track the flow of the data downstream.

# Where does this tool fit in?
dbt can be considered akin to `configuration as code` libraries. Except, here we are configuring our databse model as code. But why is that powerful? When there are many users in your database, it can be difficult for the DE team to track and maintain every table downstream of the raw data, while remaining flexible enough to allow other SQL-fluent users and developers to contribute. Dbt can allow a DE team to maintain a central repository for database objects that is (almost) pure SQL - an accesible entry point for other teams and users to create pull requests or issues.  

# Is there a quickstart guide?
[dbt has a great tutorial](https://docs.getdbt.com/tutorial/setting-up/). However, they use a cloud-hosted BigQuery db and don't give you any options for getting the data into BQ. So those unfamiliar with BQ may prefer to have something to test locally, like a Postgres db. Luckily, I've put together a [handy repo](https://github.com/Ecalzo/dbt_tutorial) for quickly getting a local Postgres db up and running with raw data from the dbt tutorial.

# Is dbt a one-size-fits-all solution?
Nope. There are definitely use cases where dbt can be a hindrance or require orchestration with a tool like Airflow. Consider this example: You would like to model some data, then feed that modeled data into your data science model, which is in python. The output from the model needs to then populate some tables, which get combined with the first round of modeled data. This is the final state. So your data science model is dependent on dbt running half way, pausing to let the model run on some transformed data, then completing all downstream database transformations with the latest data science output. This requires orchestration, and will not be accomplished with a simple `dbt run` command. Scenarios like this are worth keeping in mind when considering dbt for your use case.
