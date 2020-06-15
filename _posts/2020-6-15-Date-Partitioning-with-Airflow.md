---
layout: post
title: Date Partitioning with Airflow
---

How can we quickly partition data by date in something like an S3 Bucket?

A common way to do this is with a function that returns a date partitioned file path based on the execution date of the DAG. In this way the dag can be made idempotent, meaning that dag runs of the same execution_date produce the same result every time.

{% highlight python %}
from airflow.operators.python_operator import PythonOperator
from airflow.macros import ds_format

dag = DAG(...)


def get_date_part(ds):
    return ds_format(ds, '%Y-%m-%d', '%Y/%m/%d/')


def write_to_s3_bucket(ds):
    date_part = get_date_part(ds)
    file_path = os.path.join(date_part, 'my_file.csv')


write_to_bucket = PythonOperator(
    task_id='write_to_bucket',
    python_callable=write_to_s3_bucket,
    # the {{ ds }} macros corresponds to the dag's execution date
    op_kwargs={'ds': '{{ ds }}'},
    dag=dag
)

{% endhighlight %}
