---
title: CREATE CHANGEFEED
summary: The CREATE CHANGEFEED statement creates a new changefeed, which provides row-level change subscriptions.
toc: true
---

<span class="version-tag">New in v2.1:</span> The `CREATE CHANGEFEED` [statement](sql-statements.html) creates a new changefeed, which provides row-level change subscriptions.

Changefeeds target a whitelist of databases and tables, called the "watched rows." Every change to a watched row is emitted as a record in a configurable format (`JSON`) to a configurable sink ([Kafka](https://kafka.apache.org/)).

For more information, see [Change Data Capture](change-data-capture.html).

{{site.data.alerts.callout_danger}}
**This feature is under active development** and only works for a targeted a use case. Please [file a Github issue](file-an-issue.html) if you have feedback on the interface.

In v2.1, CDC with Kafka is an enterprise feature. There will have a core version.
{{site.data.alerts.end}}

## Required privileges

Changefeeds can only be created by superusers, i.e., members of the `admin` role. The admin role exists by default with `root` as the member.

## Synopsis

~~~

CREATE CHANGEFEED FOR TABLE <table_name> INTO <location...>
       [ WITH <option> [= <value>] [, ...] ]

Options:
    UPDATED
    RESOLVED
    envelope=[key_only|row]
    cursor=<timestamp>
    format=json
~~~

## Parameters

Parameter | Description
----------|------------
`table_name` | The name of the table to create a changefeed for.
`location` | The location of the configurable sink. The scheme of the URI indicates the type; currently, only `kafka`. There are query parameters that vary per type. Currently, the `kafka` scheme only has `topic_prefix`, which adds a prefix to all of the topic names.<br><br>Sink URI scheme: `'[scheme]://[host]:[port][?topic_prefix=[foo]]'` <br><br>For example, `CREATE CHANGEFEED FOR TABLE foo INTO 'kafka://...?topic_prefix=bar_'` would emit rows under the topic `bar_foo` instead of `foo`.
`UPDATED` | Include updated timestamps in records.
`RESOLVED` | Periodically emit resolved timestamps to the changefeed.
`envelope=[key_only|row]` | Use `key_only` to emit only the key and no value, which is faster if you only want to know when the key changes.<br><br>Default: `WITH envelope=row `
`cursor=<timestamp>` | Emits any changes after the given [timestamp](timestamp.html), but does not output the current state of the table first. If `cursor` is not specified, the changefeed starts by doing a consistent scan of all the watched rows and emits the current value, then moves to emitting any changes that happen after the scan.<br><br>Example: `cursor=TIMESTAMP '2018-08-30 12:30:02.00-05:00'`
`format=json` | _(Default)_ Format of the emitted record.

<!-- `IF NOT EXISTS` | Create a new changefeed only if a changefeed of the same name does not already exist; if one does exist, do not return an error.
`name` | The name of the changefeed to create, which [must be unique](#create-fails-name-already-in-use) and follow these [identifier rules](keywords-and-identifiers.html#identifiers).
`WITH envelope=key_only` | Emits only the key and no value, which is faster if you only want to know when the key changes. `WITH envelope=row `is the default. In v2.1, there will also be a `WITH envelope=diff`, which emits the old and new value of the changed row.
`WITH format=json` | Default value. In v2.1, `WITH format=avro` will also be supported.-->

## Examples

### Create a changefeed

{{site.data.alerts.callout_info}}
When used with Kafka, `CREATE CHANGEFEED` is an enterprise feature.
{{site.data.alerts.end}}

{% include copy-clipboard.html %}
~~~ sql
> CREATE CHANGEFEED FOR TABLE name INTO 'kafka://host:port' WITH UPDATED, RESOLVED;
~~~
~~~
+--------------------+
|       job_id       |
+--------------------+
| 360645287206223873 |
+--------------------+
(1 row)
~~~

For for information on how to create a changefeed connected to Kafka, see [Change Data Capture](change-data-capture.html#create-a-changefeed-connected-to-kafka).

### Manage a changefeed

Use the following SQL statements to pause, resume, and cancel a changefeed.

{{site.data.alerts.callout_info}}
Changefeed-specific SQL statements (e.g., `CANCEL CHANGEFEED`) will be added in the future.
{{site.data.alerts.end}}

#### Pause a changefeed

{% include copy-clipboard.html %}
~~~ sql
> PAUSE JOB job_id;
~~~

For more information, see [`PAUSE JOB`](pause-job.html).

#### Resume a paused changefeed

{% include copy-clipboard.html %}
~~~ sql
> RESUME JOB job_id;
~~~

For more information, see [`RESUME JOB`](resume-job.html).

#### Cancel a changefeed

{% include copy-clipboard.html %}
~~~ sql
> CANCEL JOB job_id;
~~~

For more information, see [`CANCEL JOB`](cancel-job.html).

## See also

- [Change Data Capture](change-data-capture.html)
- [Other SQL Statements](sql-statements.html)
