---
title: Database Configuration
menu:
  influxdb_013:
    weight: 60
    parent: administration
---

The InfluxDB configuration file contains configuration settings specific to a local node.

## Using Configuration Files

The `influxd config` command will print out a new TOML-formatted configuration with all the available configuration options set to their default values.
On POSIX systems, a new configuration file can be generated by redirecting the output of the command to a file.

```bash
influxd config > /etc/influxdb/influxdb.generated.conf
```

Custom settings from an older configuration file can be preserved when generating a new config file using the `-config` option.

```bash
influxd config -config /etc/influxdb/influxdb.conf.old > /etc/influxdb/influxdb.conf.new
```

There are two ways to launch InfluxDB with your configuration file:

* Point the process to the correct configuration file by using the `-config`
option:

    ```bash
    influxd -config influxdb.generated.conf
    ```
* Set the environment variable `INFLUXDB_CONFIG_PATH` to the path of your
configuration file and start the process.
For example:

    ```
    echo $INFLUXDB_CONFIG_PATH
    /root/influxdb.generated.conf

    influxd
    ```

InfluxDB first checks for the `-config` option and then for the environment
variable.
If you do not supply a configuration file, InfluxDB uses an internal default
configuration (equivalent to the output of `influxd config`).
A new configuration file should be generated every time InfluxDB is upgraded.

See the [installation documentation](/influxdb/v0.13/introduction/installation/#generate-a-configuration-file) for more detail on generating and using configuration files.

## Environment variables

Set any configuration option with an environment variable.
When the process launches, InfluxDB checks for environment variables of the
form below.
The environment variable overrides the equivalent option in the configuration
file.

```
INFLUXDB_<CONFIG_SECTION_NAME>_<OPTION_NAME>
```

Example:

InfluxDB will use `~/mydataz` as the
[data directory](#dir-var-lib-influxdb-data):
```
$ echo $INFLUXDB_DATA_DIR
~/mydataz
```

> **Note:** Replace any dashes in option names with underscores. For example,
this is the valid environment variable for the `store-database` option in the
`[monitor]` section:
>
```
INFLUXDB_MONITOR_STORE_DATABASE
```

## Configuration Sections

* [Global Options](#global-options)
* [[meta]](#meta)
* [[data]](#data)
* [[cluster]](#cluster)
* [[retention]](#retention)
* [[shard-precreation]](#shard-precreation)
* [[admin]](#admin)
* [[monitor]](#monitor)
* [[subscriber]](#subscriber)
* [[http]](#http)
* [[[graphite]]](#graphite)
* [[[collectd]]](#collectd)
* [[[opentsdb]]](#opentsdb)
* [[[udp]]](#udp)
* [[continuous_queries]](#continuous-queries)

## Configuration Options

Every configuration section has configuration options.
Every configuration option is optional.
If a configuration option is not provided, its default value will be used.
All configuration options listed below are set to their default value.

>**Note:** This page documents configuration options for the latest official release - the [sample configuration file on GitHub](https://github.com/influxdb/influxdb/blob/master/etc/config.sample.toml) will always be slightly ahead of what is documented here.

## Global Options

### reporting-disabled = false

InfluxData, the company, relies on reported data from running nodes
primarily to track the adoption rates of different InfluxDB versions.
This data helps InfluxData support the continuing development of
InfluxDB.  InfluxData does not request, track, or store the IP
addresses of reporting servers.

The `reporting-disabled` option toggles
the reporting of anonymous data every 24 hours to `m.influxdb.com`.
Each report includes a unique, randomly-generated identifier
(an 8-byte Raft ID), OS, architecture, InfluxDB version, and the
number of [databases](/influxdb/v0.13/concepts/glossary/#database),
[measurements](/influxdb/v0.13/concepts/glossary/#measurement), and
unique [series](/influxdb/v0.13/concepts/glossary/#series).  Setting
this option to `true` will disable reporting.

## [meta]

This section controls parameters for InfluxDB's metastore,
which stores information on users, databases, retention policies, shards, and
continuous queries.

### dir = "/var/lib/influxdb/meta"

The `meta` directory.
Files in the `meta` directory include `meta.db`.

>**Note:** The default directory for OS X installations is `/Users/<username>/.influxdb/meta`

### retention-autocreate = true

Retention policy auto-creation automatically creates a [`default` retention policy](/influxdb/v0.13/concepts/glossary/#retention-policy-rp) when a database is created.
The retention policy is named `default`, has an infinite duration, and is also set as the database's `DEFAULT` retention policy, which is used when a write or query does not specify a retention policy.
Disable this setting to prevent the creation of a `default` retention policy when creating databases.

### logging-enabled = true

Meta logging toggles the logging of messages from the meta service.

### pprof-enabled = false

### lease-duration = "1m0s"

The default duration for leases.

## [data]

This section controls where the actual shard data for InfluxDB lives and how it is flushed from the WAL. `dir` may need to be changed to a suitable place for you system, but the WAL settings are an advanced configuration. The defaults should work for most systems.

### dir = "/var/lib/influxdb/data"

The directory where InfluxDB stores the data.
This directory may be changed.

>**Note:** The default directory for OS X installations is `/Users/<username>/.influxdb/data`

### engine = "tsm1"

The storage engine. This is the only option for version 0.13.

### wal-dir = "/var/lib/influxdb/wal"

The WAL directory is the location of the [write ahead log](/influxdb/v0.13/concepts/glossary/#wal-write-ahead-log).

### wal-logging-enabled = true

The WAL logging enabled toggles the logging of WAL operations such as WAL
flushes to disk.

### query-log-enabled = true

The query log enabled setting toggles the logging of parsed queries before execution.
Very useful for troubleshooting, but will log any sensitive data contained within a query.

### cache-max-memory-size = 524288000

The cache maximum memory size is the maximum size (in bytes) a shard's cache can reach before it starts rejecting writes.

### cache-snapshot-memory-size = 26214400

The cache snapshot memory size is the size at which the engine will snapshot the cache and write it to a TSM file, freeing up memory.

### cache-snapshot-write-cold-duration = "1h0m0s"

The cache snapshot write cold duration is the length of time at which the engine will snapshot the cache and write it to a new TSM file if the shard hasn't received writes or deletes.

### compact-full-write-cold-duration = "24h0m0s"

The compact full write cold duration is the duration at which the engine will compact all TSM files in a shard if it hasn't received a write or delete.

### max-points-per-block = 0

The maximum points per block is the maximum number of points in an encoded block in a TSM file.
Larger numbers may yield better compression but could incur a performance penalty when querying.

### data-logging-enabled = true

## [cluster]

This section contains configuration options for query management.
For more on managing queries, see [Query Management](/influxdb/v0.13/troubleshooting/query_management/).

### force-remote-mapping = false

### write-timeout = "10s"

The time within which a write request must complete on the cluster.

### shard-writer-timeout = "5s"

The time within which a remote shard must respond to a write request.

### max-remote-write-connections = 3

### shard-mapper-timeout = "5s"

### max-concurrent-queries = 0

The maximum number of running queries allowed on your instance.
The default setting (`0`) allows for an unlimited number of queries.

### query-timeout = "0"

The maximum time for which a query can run on your instance before InfluxDB
kills the query.
The default setting (`0`) allows queries to run with no time restrictions.
This setting is a [duration literal](/influxdb/v0.13/query_language/spec/#durations).

### log-queries-after = "0"

The maximum time a query can run after which InfluxDB logs the query with a
`Detected slow query` message.
The default setting (`"0"`) will never tell InfluxDB to log the query.
This setting is a
[duration literal](/influxdb/v0.13/query_language/spec/#durations).

### max-select-point = 0

The maximum number of [points](/influxdb/v0.13/concepts/glossary/#point) that a
`SELECT` statement can process.
The default setting (`0`) allows the `SELECT` statement to process an unlimited
number of points.

### max-select-series = 0

The maximum number of [series](/influxdb/v0.13/concepts/glossary/#series) that a
`SELECT` statement can process.
The default setting (`0`) allows the `SELECT` statement to process an unlimited
number of series.

### max-select-buckets = 0

The maximum number of `GROUP BY time()` buckets that a query can process.
The default setting (`0`) allows a query to process an unlimited number of
buckets.

## [retention]

This section controls the enforcement of retention policies for evicting old data.

### enabled = true

Set to `false` to prevent InfluxDB from enforcing retention policies.

### check-interval = "30m0s"

The rate at which InfluxDB checks to enforce a retention policy.

## [shard-precreation]

Controls the precreation of shards so that shards are available before data arrive.
Only shards that, after creation, will have both a start- and end-time in the future are ever created.
Shards that would be wholly or partially in the past are never precreated.

### enabled = true

### check-interval = "10m0s"

### advance-period = "30m0s"

The maximum period in the future for which InfluxDB precreates shards.
The `30m` default should work for most systems.
Increasing this setting too far in the future can cause inefficiencies.

## [admin]

Controls the availability of the built-in, web-based admin interface.

### enabled = true

Set to `false` to disable the admin interface.

### bind-address = ":8083"

The port used by the admin interface.

### https-enabled = false

Set to `true` to enable HTTPS for the admin interface.

>**Note:** HTTPS must be enable for the [[http]](/influxdb/v0.13/administration/config/#http) service for the admin UI to function properly using HTTPS.

### https-certificate = "/etc/ssl/influxdb.pem"

The path of the certificate file.

### Version = ""

## [monitor]

This section controls InfluxDB's [system self-monitoring](https://github.com/influxdb/influxdb/blob/master/monitor/README.md).

By default, InfluxDB writes the data to the `_internal` database.
If that database does not exist, InfluxDB creates it automatically.
The `DEFAULT` retention policy on the `_internal` database is seven days.
If you want to use a retention policy other than the seven-day retention policy, you must [create](/influxdb/v0.13/query_language/database_management/#retention-policy-management) it.

### store-enabled = true

Set to `false` to disable recording statistics internally.
If set to `false` it will make it substantially more difficult to diagnose issues with your installation.

### store-database = "\_internal"

The destination database for recorded statistics.

### store-interval = "10s"

The interval at which InfluxDB records statistics.

## [subscriber]

### enabled = true

## [http]

This section controls how InfluxDB configures the HTTP endpoints.
These are the primary mechanisms for getting data into and out of InfluxDB.
Edit the options in this section to enable HTTPS and authentication.
See [Authentication and Authorization](/influxdb/v0.13/administration/authentication_and_authorization/).

### enabled = true

Set to `false` to disable HTTP.
Note that the InfluxDB [command line interface (CLI)](/influxdb/v0.13/tools/shell/) connects to the database using the HTTP API.

### bind-address = ":8086"

The port used by the HTTP API.

### auth-enabled = false

Set to `true` to require authentication.

### log-enabled = true

Set to `false` to disable logging.

### write-tracing = false

Set to `true` to enable logging for the write payload.
If set to `true`, this will duplicate every write statement in the logs and is thus not recommended for general use.

### pprof-enabled = false

Set to `true` to enable [pprof](http://blog.golang.org/profiling-go-programs) on InfluxDB so that it gathers detailed performance information.

### https-enabled = false

Set to `true` to enable HTTPS.

### https-certificate = "/etc/ssl/influxdb.pem"

The path of the certificate file.

### max-row-limit = 10000

## [[graphite]]

This section controls one or many listeners for Graphite data.
See the [README](https://github.com/influxdb/influxdb/blob/master/services/graphite/README.md) on GitHub for more information.

### enabled = false

Set to `true` to enable Graphite input.

### bind-address = ":2003"

The default port.

### database = "graphite"

The name of the database that you want to write to.

### protocol = "tcp"

Set to `tcp` or `udp`.

*The next three options control how batching works.
You should have this enabled otherwise you could get dropped metrics or poor performance.
Batching will buffer points in memory if you have many coming in.*

### batch-size = 5000

The input will flush if this many points get buffered.

### batch-pending = 10

The number of batches that may be pending in memory.

### batch-timeout = "1s"

The input will flush at least this often even if it hasn't reached the configured batch-size.

### consistency-level = "one"

The number of nodes that must confirm the write.
If the requirement is not met the return value will be either `partial write` if some points in the batch fail or `write failure` if all points in the batch fail.
For more information, see the Query String Parameters for Writes section in the [Line Protocol Syntax Reference ](/influxdb/v0.13/write_protocols/write_syntax/).

### separator = "."

This string joins multiple matching 'measurement' values providing more control over the final measurement name.

### udp-read-buffer = 0

UDP Read buffer size, 0 means OS default.
UDP listener will fail if set above OS max.

## [[collectd]]

This section controls the listener for collectd data. See the
[README](https://github.com/influxdata/influxdb/tree/master/services/collectd)
on Github for more information.

### enabled = false

Set to `true` to enable collectd writes.

### bind-address = ":25826"

The port.

### database = "collectd"

The name of the database that you want to write to.
This defaults to `collectd`.

*The next three options control how batching works.
You should have this enabled otherwise you could get dropped metrics or poor performance.
Batching will buffer points in memory if you have many coming in.*

### batch-size = 5000

The input will flush if this many points get buffered.

### batch-pending = 10

The number of batches that may be pending in memory.

### batch-timeout = "10s"

The input will flush at least this often even if it hasn't reached the configured batch-size.

### read-buffer = 0

UDP Read buffer size, 0 means OS default.
UDP listener will fail if set above OS max.

### typesdb = "/usr/share/collectd/types.db"

Defaults to `/usr/share/collectd/types.db`. A sample `types.db` file
can be found
[here](https://github.com/collectd/collectd/blob/master/src/types.db).

## [[opentsdb]]

Controls the listener for OpenTSDB data.
See the [README](https://github.com/influxdb/influxdb/blob/master/services/opentsdb/README.md) on GitHub for more information.

### enabled = false

Set to `true` to enable openTSDB writes.

### bind-address = ":4242"

The default port.

### database = "opentsdb"

The name of the database that you want to write to.
If the database does not exist, it will be created automatically when the input is initialized.

### retention-policy = ""

The relevant retention policy.
An empty string is equivalent to the database's `DEFAULT` retention policy.

### consistency-level = "one"

### tls-enabled = false

### certificate = "/etc/ssl/influxdb.pem"

*The next three options control how batching works.
You should have this enabled otherwise you could get dropped metrics or poor performance.
Only points metrics received over the telnet protocol undergo batching.*

### batch-size = 1000

The input will flush if this many points get buffered.

### batch-pending = 5

The number of batches that may be pending in memory.

### batch-timeout = "1s"

The input will flush at least this often even if it hasn't reached the configured batch-size.

### log-point-errors = true

Log an error for every malformed point.

## [[udp]]

This section controls the listeners for InfluxDB line protocol data via UDP.
See the [UDP page](/influxdb/v0.13/write_protocols/udp/) for more information.

### enabled = false

Set to `true` to enable writes over UDP.

### bind-address = ":8089"

An empty string is equivalent to `0.0.0.0`.

### database = "udp"

The name of the database that you want to write to.

### retention-policy = ""

The relevant retention policy for your data.
An empty string is equivalent to the database's `DEFAULT` retention policy.

*The next three options control how batching works.
You should have this enabled otherwise you could get dropped metrics or poor performance.
Batching will buffer points in memory if you have many coming in.*

### batch-size = 5000

The input will flush if this many points get buffered.

### batch-pending = 10

The number of batches that may be pending in memory.

### batch-timeout = "1s"

The input will flush at least this often even if it hasn't reached the configured batch-size.

### read-buffer = 0

UDP read buffer size, 0 means OS default.
UDP listener will fail if set above OS max.

### precision = ""

## [continuous_queries]

This section controls how [continuous queries (CQs)](/influxdb/v0.13/concepts/glossary/#continuous-query-cq) run within InfluxDB.
CQs are automated batches of queries that execute over recent time intervals.
InfluxDB executes one auto-generated query per `GROUP BY time()` interval.

### log-enabled = true

Set to `false` to disable logging for CQ events.

### enabled = true

Set to `false` to disable CQs.

### run-interval = "1s"

The interval at which InfluxDB checks to see if a CQ needs to run. Set this option to the lowest interval at which your CQs run. For example, if your most frequent CQ runs every minute, set `run-interval` to `1m`.
