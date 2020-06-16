---
layout: post
title: Idempotency - Why it matters
---

## What?
In DE, Idempotency is the idea that a single ETL job or process will produce the same end result regardless of how many times you re-run the job. That means that if you have a DAG that runs on 6/15/2020, then if you clear and run that DAG 1000x, your data warehouse will still hold the exact same data, no duplicates. **This concept is extremely important and will save you time in the long run**. 

Combining this idea with Airflow can be fairly easy with some knowledge of your dataset, let's take a look:

## In Airflow

{% highlight python %}
# imports ommitted for brevity

dag = DAG (...)

def get_delete_statement(target_table, date):
    query = """
        DELETE FROM {target_table}
        WHERE DATE = '{date}'
    """

    return query.format(target_table=target_table, date=date)


def get_copy_into_statement(target_table, date):
    # copying from S3 varies by database
    # this is a gross simplification
    query = """
        COPY INTO {target_table}
        FROM s3://my_bucket/{date}/my_file.csv
    """
    return query.format(target_table=target_table, date=date)


def delete_and_write_records(ds):
    engine = sa.create_engine(...)
    target_table = 'my_schema.my_table'
    date = ds_format(ds, '%Y-%m-%d', '%Y/%m/%d')

    copy_statement =  get_copy_into_statement(target_table, date)
    delete_statement = get_delete_statement(target_table, date)

    with engine.begin() as conn:
        # first we delete records for that date
        conn.execute(delete_statement)
        # then replace those records with data for that date
        conn.execute(copy_statement)


delete_and_write = PythonOperator(
    task_id='delete_and_write',
    python_callable=delete_and_write_records,
    op_kwargs={'ds': '{{ "{{ ds " }}}}'},
    dag=dag
)

{% endhighlight %}

## Summary
I hope this short sample DAG helps to outline an idempotent ETL process using Airflow. Since this DAG is dependent on the execution_date, we can run it a few hundred times and produce the same results (as long as the S3 file remains the same). Incorporating idempotency into your ETL processes works will let you quickly rerun failed jobs or backfill.

Thanks for reading.
