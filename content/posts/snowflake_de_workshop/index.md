---
sys:
  pageId: "2bd1f336-f350-8085-891c-cb21c6f14a12"
  createdTime: "2025-12-02T07:40:00.000Z"
  lastEditedTime: "2025-12-05T03:08:00.000Z"
  propFilepath: "posts/snowflake_de_workshop/index.md"
title: "Snowflake's Data Engineering Workshop"
date: "2025-12-05T00:00:00Z"
description: ""
tags:
  - "❄Snowflake"
  - "🖇Data Engineering"
categories:
  - "📱 技術"
author: "Harry Yang"
draft: false
url: "/posts/snowflake-de-workshop"
section: "posts"
---

# Hop on the Journey


I'm thrilled to share that I've completed the Snowflake Badge 5: Data Engineering Workshop, marking the enriching journey on Snowflake's ecosystem. This workshop is a fast-paced, interactive course that gives data engineering fundamentals on the Snowflake platform, covering topics like data loading with [Snowpipe](https://www.google.com/search?q=Snowpipe&sca_esv=3fb0e2d94fbc189e&ei=Q_wvaYDNJ-iPvr0PkLXr0Qs&ved=2ahUKEwj9-vSuh6GRAxVdja8BHX9IINAQgK4QegQIARAE&uact=5&oq=Hands-On%20Essentials%3A%20Data%20Engineering%20Workshop%20by%20snowflake%20brief%20introduction&gs_lp=Egxnd3Mtd2l6LXNlcnAiTkhhbmRzLU9uIEVzc2VudGlhbHM6IERhdGEgRW5naW5lZXJpbmcgV29ya3Nob3AgYnkgc25vd2ZsYWtlIGJyaWVmIGludHJvZHVjdGlvbkj9YlCmAVibYHADeACQAQCYAZYBoAHXFKoBBDMxLjS4AQPIAQD4AQGYAhKgAskKwgIKEAAYsAMY1gQYR8ICBhAAGBYYHsICBRAAGO8FwgIIEAAYgAQYogTCAgUQIRigAcICCBAAGKIEGIkFwgIHECEYoAEYCsICBBAhGAqYAwCIBgGQBgiSBwQxNS4zoAfMXLIHBDEzLjO4B8EKwgcGNS4xMi4xyAcb&sclient=gws-wiz-serp&mstk=AUtExfDmQmmwSBWfFO43Zi2W5wTo_0apwQ1qPnZ4izuBUPUuyJv3uSooEyhzR-EEdgHWPh4atahgvyqdq_3I3MqpE8MhRkQPjFCEJWD13lMKOztw8lPUpE_6lkvirvtXhCpp01MLpjk7kbvAHQv_xj66ORdvWQoYWX3ho-exPX6kI9STuTIHfFUaOsLqC-w2ZEdXgos1&csui=3), data transformation with SQL, task scheduling and more. The goal is to provide practical experience in data management for data-driven organizations and earn a badge upon completion.


The best thing about it is that it is a hands-on course so you don’t have to worried if you are not familiar to Snowflake and you know what it is also free on [https://learn.snowflake.com/en/pages/hands-on-essentials-track/](https://learn.snowflake.com/en/pages/hands-on-essentials-track/).


The journey began from data ingestion to landing on a simple dashboard. In Snowflake you can directly loading data from cloud storage services like Amazon S3, Google Cloud Storage, and Microsoft Azure by leveraging bulk or continuous loading.


However, having the raw data loaded is not well-prepared for users to access. Therefore, before transferring data, engineers need to understand the requirement and then analyze the data in order to find solutions to build the pipeline.


This workshop provides a scenario where the time zone is missing, but which is a critical geographical information for user base. So after the data is fed into a raw table, data need to be converted or merge with other tables provided with geographic information. These steps are so called transformation in data engineering.


After the user requirements are fulfilled, the next step is to optimize the pipeline in order to **reduce costs** spent on computing units. Instead of checking the need for feeding data time by time, it is better to load data only if a change happened in the source. In other word, the goal is to make the pipeline more responsive. Hence publish and subscribe services and message queue systems are introduced to replace the original time-triggered data loading and transformation tasks.


Here are some key notes that I’ve learned from this course. If you are interested, take a look and have fun building better data pipelines!


# Merge Insert VS Insert


The core difference between the query `INSERT` and `INSERT MERGE` lies in how they handle existing data in the target table.


| Query                    | `INSERT`                                                                                                                                                          | `INSERT MERGE`                                                                                                       |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Operation Type**       | Unconditional Append                                                                                                                                              | Conditional Update                                                                                                   |
| **Data Handling**        | Always adds _new_ rows from the source query to the target table, even if the exact same event was already present.                                               | Checks if a record from the source query _already exists_ in the target table based on specific join conditions.     |
| **Duplicate Prevention** | Does not prevent duplicates by itself. Running the task multiple times without clearing the target table would result in duplicate rows for the same game events. | Prevents duplicates for records that satisfy the `ON` condition (`GAMER_NAME`, `GAME_EVENT_UTC`, `GAME_EVENT_NAME`). |
| **Target Table State**   | Target table likely grows with duplicate data upon re-execution.                                                                                                  | Target table remains clean; only truly _new_ unique records are added.                                               |


**Summary of Differences**


1. The `INSERT` Approach (Original)


A simple `INSERT INTO ... SELECT` DML always adds new rows from the source query to the target table, even if the exact same data was already present. This could be problematic because the source table likely contains historical data that doesn't get cleared immediately. Each time the task runs, it re-processes all rows from source data and inserts them again into target table, creating many duplicate entries.


2. The `INSERT MERGE` Approach (Second Query)


The second task uses `INSERT MERGE` to synchronize two tables by checking if a record from the source already existed in the target table based on join conditions. `INSERT MERGE` is idempotent which can be run multiple times without causing duplicates. It uses a `WHEN NOT MATCHED THEN INSERT` clause to effectively synchronizes the target table, preventing duplicates for records that satisfy the `ON` condition.


**Conclusion**


The `INSERT MERGE` manipulation is better for maintaining a clean, non-duplicated target table that acts as a historical record of events. Using only `INSERT` is generally used only when the source data is entirely new.


# Use Serverless resource to run tasks & use task dependency to increase stability


### TASK DEPENDENCIES


At a first look, our pipeline design grants us full control over the data loading and transformation processes, but our loading and transformation tasks currently run on schedule and independently. This independent execution introduces uncertainty regarding whether the preceding task has truly produced the necessary data. Implementing task dependencies can eliminate this issue. What need to do is replacing the time-based `SCHEDULE` by configuring the transformation tasks to only run upon the successful completion of their respective load jobs.


### SERVERLESS COMPUTE


The WAREHOUSE that are using to run the tasks has to spin up each time the tasks are triggered. Then, if it's designed to auto-suspend in 5 minutes, it won't EVER suspend, because the task will run again before it has time to shut down. This can cost a lot of credits of computation units which means spending more unnecessary costs.


Snowflake has a different option called "SERVERLESS". It means that tasks don't have to spin up a warehouse, instead it can use a thread or two of another compute resource that is already running. Serverless compute is much more efficient for those small tasks that don't do very much, but do what they do quite often.


To use the SERVERLESS task mode, we'll need to grant that privilege to SYSADMIN.


{{< alert "lightbulb" >}}
Grant Serverless Task Management to SYSADMIN


```sql
use role accountadmin;
grant EXECUTE MANAGED TASK on account to SYSADMIN;

--switch back to sysadmin
use role sysadmin;
```


{{< /alert >}}


Combing these two improvements, all need to is

- Replace the WAREHOUSE Property in Your Tasks
- Remove the SCHEDULE property and have `LOAD_LOGS_ENHANCED` run each time GET_NEW_FILES completes

```sql
create or replace task AGS_GAME_AUDIENCE.RAW.LOAD_LOGS_ENHANCED
	USER_TASK_MANAGED_INITIAL_WAREHOUSE_SIZE='XSMALL'
	after AGS_GAME_AUDIENCE.RAW.GET_NEW_FILES
	as MERGE INTO AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED e
    USING (
        SELECT logs.ip_address
        , logs.user_login as GAMER_NAME
        , logs.user_event as GAME_EVENT_NAME
        , logs.datetime_iso8601 as GAME_EVENT_UTC
        , city
        , region
        , country
        , timezone as GAMER_LTZ_NAME
        , CONVERT_TIMEZONE('UTC', timezone, logs.datetime_iso8601) as game_event_ltz
        , DAYNAME(game_event_ltz) as DOW_NAME
        , tod_name
        from AGS_GAME_AUDIENCE.RAW.PL_LOGS logs
        JOIN IPINFO_GEOLOC.demo.location loc
        ON IPINFO_GEOLOC.public.TO_JOIN_KEY(logs.ip_address) = loc.join_key
        AND IPINFO_GEOLOC.public.TO_INT(logs.ip_address)
        BETWEEN start_ip_int AND end_ip_int
        JOIN ags_game_audience.raw.time_of_day_lu tod
        ON HOUR(game_event_ltz) = tod.hour
) r
ON r.gamer_name = e.GAMER_NAME
and r.game_event_utc = e.game_event_utc
and r.game_event_name = e.game_event_name
WHEN NOT MATCHED THEN
INSERT (IP_ADDRESS, GAMER_NAME, GAME_EVENT_NAME, GAME_EVENT_UTC, CITY, REGION, COUNTRY, GAMER_LTZ_NAME, GAME_EVENT_LTZ, DOW_NAME, TOD_NAME)
VALUES (IP_ADDRESS, GAMER_NAME, GAME_EVENT_NAME, GAME_EVENT_UTC, CITY, REGION, COUNTRY, GAMER_LTZ_NAME, GAME_EVENT_LTZ, DOW_NAME, TOD_NAME);
```


# From time-driven pipeline to event-driven pipeline


In modern data pipelines building, it is more than creating a Task-Driven or Time-Driven Pipeline like the picture showing below. These tasks could be built into fewer pieces for easier maintenance and leveraging cloud services offered by major cloud providers to create a  responsive event-driven pipeline.


{{< figure href="https://www.notion.so/image/https%3A%2F%2Flearn.snowflake.com%2Fassets%2Fcourseware%2Fv1%2F77498dce39b23c81cd56ded4c0883cf2%2Fasset-v1%3Asnowflake%2BESS-DNGW%2BA%2Btype%40asset%2Bblock%2FDNGW_112.png?table=block&id=2c01f336-f350-803b-be52-c5a6d07762e5&spaceId=b30abf31-1973-404d-9af0-238ebdc05988&width=2000&userId=f2d1f992-8d41-4465-8925-5b0d1efea660&cache=v2" class="mx-auto" caption="Origin Pipeline Design" >}}


That being said step 2 and step 3 are both essentially a data loading task, so they can be integrated as simple loading task that directly extract basic information from the source data. Furthermore, running this task on a planned schedule is not quite smart and efficient. Snowflake offers a service called Snowpipe that can leverage cloud engineering infrastructure objects to make continuous loading possible. This data pipeline will then be built like the DAG below which incorporate Publish and Subscribe services and Message Queueing system.


{{< figure href="https://learn.snowflake.com/assets/courseware/v1/b4526d45cc9e012d57395b918523aed5/asset-v1:snowflake+ESS-DNGW+A+type@asset+block/DNGW_115.png" class="mx-auto" caption="Event-based Pipeline Design" >}}


Publish and Subscribe services are based on a Hub and Spoke pattern. The HUB is a central controller that manages the receiving and sending of messages. The SPOKES are the solutions and services that either send or receive notifications from the HUB.


If a SPOKE is a PUBLISHER, that means they send messages to the HUB. If a SPOKE is a SUBSCRIBER, that means they receive messages from the HUB. A SPOKE can be both a publisher and a subscriber.


With messages flowing into and out of a Pub/Sub service from so many places, it could get confusing, fast. So Pub/Sub services have EVENT NOTIFICATIONS and TOPICS. A topic is a collection of event types. A SPOKE publishes NOTIFICATIONS to a TOPIC and subscribes to a TOPIC, which is a stream of NOTIFICATIONS.


{{< gallery caption="Publish and Subscribe services and Message Queueing">}}
{{< figure href="https://learn.snowflake.com/assets/courseware/v1/28e6573484e3986451bfe995ce2b3f9c/asset-v1:snowflake+ESS-DNGW+A+type@asset+block/DNGW_110.png" class="auto">}}
{{< figure href="https://learn.snowflake.com/assets/courseware/v1/4cf563ed6830d91ecdfa60e420469c04/asset-v1:snowflake+ESS-DNGW+A+type@asset+block/DNGW_132.png" class="auto">}}
{{< /gallery >}}


By making the loading data task event-driven, all need to do is create a PIPE that subscribe to the topic hosting on a cloud hub.


```sql
CREATE OR REPLACE PIPE PIPE_GET_NEW_FILES
auto_ingest=true
aws_sns_topic='arn:aws:sns:us-west-2:321463406630:dngw_topic'
AS
COPY INTO ED_PIPELINE_LOGS
FROM (
	...EXTRACT DATA
)
file_format = (format_name = ff_json_logs);
```


# Change Data Capture


Take a look back at the DAG that have been improved above. Steps 4 is still a time-driven task, but the good news is that the pipeline now has two of the most important pipeline devices in Snowflake: tasks and pipes. In order to improve the data flow further, Snowflake provides a service called a STREAM that can make it more responsive and more efficient.


STREAM will not replace the origin task but making it event-driven by using "Change Data Capture". In the simplest use, streams appear very simple. They are just a running list of things that have changed in your table or view. But in production use, to use streams effectively, you will need to understand offsets, data retention periods, staleness, and stream types.


In this workshop, it only covers the most basic use of a stream, one that will only handle record inserts and will not be guaranteed to track and process every change.


{{< figure href="https://learn.snowflake.com/assets/courseware/v1/77498dce39b23c81cd56ded4c0883cf2/asset-v1:snowflake+ESS-DNGW+A+type@asset+block/DNGW_112.png" class="mx-auto" caption="Let STREAM do the work" >}}


By running the following command to create a STREAM under raw schema that detects data changes in `AGS_GAME_AUDIENCE.RAW.ED_PIPELINE_LOGS`.


```sql
create or replace stream ags_game_audience.raw.ed_cdc_stream
on table AGS_GAME_AUDIENCE.RAW.ED_PIPELINE_LOGS;
```


The next step is to alter the origin task that runs every 5 minutes to update the enhanced table for analysis. Since the STREAM service is up, it won’t need to check the whole table to see if any data changes happened in every run. Instead, by creating a `WHEN` condition it can let the STREAM service handle it.


```sql
create or replace task AGS_GAME_AUDIENCE.RAW.CDC_LOAD_LOGS_ENHANCED
	USER_TASK_MANAGED_INITIAL_WAREHOUSE_SIZE='XSMALL'
	SCHEDULE = '5 minutes'
WHEN
    system$stream_has_data('ed_edc_stream')
	as
MERGE INTO AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED e
USING (
        ... TRANSFORM DATA
      ) r
ON r.GAMER_NAME = e.GAMER_NAME
AND r.GAME_EVENT_UTC = e.GAME_EVENT_UTC
AND r.GAME_EVENT_NAME = e.GAME_EVENT_NAME
WHEN NOT MATCHED THEN
INSERT (IP_ADDRESS, GAMER_NAME, GAME_EVENT_NAME
        , GAME_EVENT_UTC, CITY, REGION
        , COUNTRY, GAMER_LTZ_NAME, GAME_EVENT_LTZ
        , DOW_NAME, TOD_NAME)
VALUES
        (IP_ADDRESS, GAMER_NAME, GAME_EVENT_NAME
        , GAME_EVENT_UTC, CITY, REGION
        , COUNTRY, GAMER_LTZ_NAME, GAME_EVENT_LTZ
        , DOW_NAME, TOD_NAME);
```


### Data is Flowing


With the PIPE and STREAM in place, it all need is a task at the end that pulls new data from the STREAM, instead of from the RAW data table. The data now makes two stops in the flow. The first table the data lands on `ED_PIPELINE_LOGS` done by the Snowpipe named `GET_NEW_FILES`. The second stop for the data is the `AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED` table which is a work of STREAM and a scheduled task. 


### What Happens if the Merge Fails?


This is one of the reasons Streams can be very complex. Since the information disappears as you process it, when using a stream in production systems, you will likely want to have more protection in place to make sure you don't lose information. For example, our old task that looks at EVERY row EVERY time could be run just once every 24 hours so that it could pick up anything missed by the task that processes the stream.


# Final Thoughts


Overall, I enjoyed this course that gives a good overview of what is available in Snowflake. Moreover, I've learned more concepts for modern data engineering, which I'm eager to apply on a practical data engineering project. I encourage anyone who is interested in what Snowflake has to offer for data engineering to enroll in this course.
