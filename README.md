![Flight Deck](https://raw.githubusercontent.com/ten7/flight-deck/master/flightdeck-logo.png)

# Flight Deck Solr

Flight Deck Solr is a minimalist Solr container for Drupal sites on Kubernetes and Docker. You can use it both for local development and production.

Features:
* ConfigMap-friendly YAML configuration
* Supports multiple cores
* Create [Search API Solr](https://www.drupal.org/project/search_api_solr) and [Apachesolr module](https://www.drupal.org/project/apachesolr) compatible cores without providing a schema
* Supports custom cores via ConfigMaps or volumes


## Tags and versions

There are several tags available for this container, each with different Solr and Drupal module support:

| Tags | Solr Version | Search API | Custom cores |
| ---- | ------------ | ---------- | ------------ |
| latest | 8.6 | Yes | Yes |
| x.y.z | 8.6 | Yes | Yes |


## Configuration

Instead of a large number of environment variables, this container relies on a file to perform all runtime configuration, `flightdeck-solr.yml`. Inside the file, create following:

```yaml
---
flightdeck_solr: {}
```

All configuration is done as items under the `flightdeck_solr` variable. See the following sections for details as to particular configurations.

You can provide this file in one of three ways to the container:

* Mount the configuration file at path `/config/solr/flightdeck-solr.yml` inside the container using a bind mount, configmap, or secret.
* Mount the config file anywhere in the container, and set the `FLIGHTDECK_CONFIG_FILE` environment variable to the path of the file.
* Encode the contents of `flightdeck-solr.yml` as base64 and assign the result to the `FLIGHTDECK_CONFIG` environment variable.

### Basic settings

```yaml
---
flightdeck_solr:
  port: 8983
  cores: []
```

Where:
* **flightdeck_solr.port** is the port on which to run Solr. Optional, defailts to `8983`.
* **flightdeck_solr.cores** is a list of cores to create.

### Defining cores

This container supports multiple cores by defining an item under the `flightdeck_solr.cores` item:

```yaml
---
flightdeck_solr:
  port: 8983
  cores:
    - name: "my_custom_core"
      type: "custom"
      confDir: "/solr-conf"
      dataDir: "/solr-data"
    - name: "my_other_core"
      type: "custom"
      confDir: "/other-conf"
      dataDir: "/other-data"
```

Each item has the following variables:

* **name** is the name of the solr core. Required.
* **type** is the type of Solr core to create. Optional, must be `custom` or `empty`.
* **confDir** is the path inside the container at which the schema files are mounted. Required for `custom` cores.
* **dataDir** is the path inside the container at which the core data is to be stored. Optional, defaults to `/data/&lt;name_of_the_core&gt;`.

### Passing core configs inline

In some cases, you may wish to pass your solr core configs inline inside of `flightdeck-solr.yml` rather than as a mounted directory in the container. In that case, you can define the `conf` list.

Each item under the list represents a file in the core configuration directory:

```yaml
---
flightdeck_solr:
  port: 8983
  cores:
    - name: "my_custom_core"
      type: "custom"
      conf:
        - name: "solrconfig.xml"
          content: |
            <?xml version="1.0" encoding="UTF-8" ?>

            <!DOCTYPE config [
              <!ENTITY extra SYSTEM "solrconfig_extra.xml">
              <!ENTITY index SYSTEM "solrconfig_index.xml">
            ]>
```

Where:

* **name** is the name of the file to create.
* **content** is the content of the file.

### Encoding issues for inline conf

A drawback of inline configuration is that you need to escape it work in YAML. This can be frustrating an difficult. Instead, is recommended to base64 encode the content of the file, and use the `content_base64` key instead:

```yaml
---
flightdeck_solr:
  port: 8983
  cores:
    - name: "my_custom_core"
      type: "custom"
      conf:
        - name: "solrconfig.xml"
          content_base64: |
            PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiID8+Cgo8IURPQ1RZUEUgY29uZmln
            IFsKICAgPCFFTlRJVFkgZXh0cmEgU1lTVEVNICJzb2xyY29uZmlnX2V4dHJhLnhtbCI+CiAgIDwh
            RU5USVRZIGluZGV4IFNZU1RFTSAic29scmNvbmZpZ19pbmRleC54bWwiPgpdPgo=
```

## Deployment on Kubernetes

Use the [`ten7.flightdeck_cluster`](https://galaxy.ansible.com/ten7/flightdeck_cluster) role on Ansible Galaxy to deploy Solr as a statefulset:

```yaml
flightdeck_cluster:
  namespace: "search"
  configMaps:
    - name: "solr_config"
      files:
        - name: "flightdeck-solr.yml"
          content: |
            flightdeck_solr:
              cores:
                - name: "my_sapi_docker"
                  type: "search_api_solr"
  solr:
    configMaps:
      - name: "solr_config"
        path: "/config"
```

## Using with Docker Compose

Create the `flightdeck-solr.yml` file relative to your `docker-compose.yml`. Define the `solr` service mounting the file as a volume:

```yaml
version: '3'
services:
  solr:
    image: ten7/flight-deck-solr:6.6
    volumes:
      - ./flightdeck-solr.yml:/config/flightdeck-solr.yml
    ports:
      - "8003:8983"
```

## Part of Flight Deck

This container is part of the [Flight Deck library](https://github.com/ten7/flight-deck) of containers for Drupal local development and production workloads on Docker, Swarm, and Kubernetes.

Flight Deck is used and supported by [TEN7](https://ten7.com/).


## Debugging

If you need to get verbose output from the entrypoint, set `flightdeck_debug` to `true` or `yes` in the config file.

```yaml
---
flightdeck_debug: yes
```

This container uses [Ansible](https://www.ansible.com/) to perform start-up tasks. To get even more verbose output from the start up scripts, set the `ANSIBLE_VERBOSITY` environment variable to `4`.

If the container will not start due to a failure of the entrypoint, set the `FLIGHTDECK_SKIP_ENTRYPOINT` environment variable to `true` or `1`, then restart the container.

## License

Flight Deck is licensed under GPLv3. See [LICENSE](https://raw.githubusercontent.com/ten7/flight-deck/master/LICENSE) for the complete language.
