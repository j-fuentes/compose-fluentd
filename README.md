# Example: docker-compose + fluentd logging

This example shows how to use the docker fluentd log driver with docker-compose.

## How to use

### Deploy

First we need to deploy the __monitoring stuff__:

```bash
docker-compose -f monitor.yml up
```

It uses three containers: __fluentd__, __elasticsearch__ and __kibana__. All those containers use the host network stack (to make things easier).

Then we can deploy the application:

```bash
docker-compose -f application.yml up
```

This run an __apache__ container that forward the logs to the fluentd listening on localhost.

The directive `fluentd-tag: "docker.apache.{{.ID}}"` make log lines to be tagged with `docker.apache.<contianerID>`.

### Dashboard

Kibana should be listening on `http://<dockerhostip>:5601`.

The first thing to do is configure the indexs to be readed. Go to _Settings_->_Indices_ and write the pattern `fluentd-*`. Press _Create_.

After this, it is possible to create visualizations.

NOTE: it is necessary to access `http://<dockerhostip>` to generate event and populate elasticsearch.

## Configuration

The fluentd configuration file is placed in `fluentd/volumes/etc/cloudopting.conf`.

It is divided in three parts.

### `source`

Open port where to receive __docker__ input.


### `filter`

There are three filters.

The first one discard entries that aren't requests.

The second one add two fields: `log_type` `log_subtype`

The third one parses the log string and extracts the fields `http_status` and `request_ip`


### `match`

Set the output (__elasticsearch__).

## Things that can be improved

### Datatypes mapping from fluentd to elasticsearch

Since we are using the [docker log driver](http://docs.docker.com/engine/reference/logging/fluentd/), we cannot use [parser plugins](http://docs.fluentd.org/articles/parser-plugin-overview) to extract the information from the log. This is because the log is received encapsulated in a field called `log` and alongside other fields that docker insert. It is possible to create a new plugin that do the work, but the default are useless for us.

Currently, all data parsed with [filter_record_transformer](http://docs.fluentd.org/articles/filter_record_transformer) is mapped as string. This could be an inconvenience for performance in data analysis in kibana.


## References:

- http://www.fluentd.org/guides/recipes/docker-logging
- https://github.com/fluent/fluentd-docker-image
- https://www.digitalocean.com/community/tutorials/how-to-centralize-your-docker-logs-with-fluentd-and-elasticsearch-on-ubuntu-14-04
- https://docs.docker.com/compose/yml/#log-driver
- http://www.fluentd.org/guides/recipes/elasticsearch-and-s3
