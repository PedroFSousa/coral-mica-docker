# Mica Docker

This repository contains the source code for building a [Mica](https://www.obiba.org/pages/products/mica/) Docker image based on the [original Mica image from OBiBa](https://github.com/obiba/docker-mica), modified to:
* install the [mica-search-es](https://github.com/obiba/mica-search-es) plugin and the [Mica Python client](https://github.com/obiba/mica-python-client) at build time;
* support the use of Docker secrets for passwords;
* connect to a MongoDB instance with enabled Access Control;
* automatically add custom:
  * forms for studies, populations and data collection events;
  * search criteria;
  * Opal credentials;
* grant 'Reader' permission to the `mica-reader` group on Mica documents;

## Building
After every commit, GitLab CI automatically builds the image and pushes it into INESC TEC's Docker Registry. The version tag of the image is determined from a string matching `##x.x.x##` on the commit message.

Alternatively, an image can be built locally by running:
`docker build -t mica-docker .`

## Setting Up
**Environment Variables**  
The following environment variables are available:

* `MICA_ADMINISTRATOR_PASSWORD` is the password for the administrator account of the Mica server;
* `MICA_ANONYMOUS_PASSWORD` is the password for the `anonymous` account (to be used by MIca Drupal);
* `OPAL_PORT_8443_TCP_ADDR` and `OPAL_PORT_8443_TCP_PORT` are the address and port of the Opal server;
* `OPAL_ADMINISTRATOR_PASSWORD` is the password for the administrator account of the Opal server;
* `AGATE_PORT_8444_TCP_ADDR` and `AGATE_PORT_8444_TCP_PORT` are the address and port of the Agate server;
* `MONGO_PORT_27017_TCP_ADDR` and `MONGO_PORT_27017_TCP_PORT` are the address and port of the MongoDB server;
* `MONGO_INITDB_ROOT_USERNAME` and `MONGO_INITDB_ROOT_PASSWORD` are the credentials Mica will use to connect to the MongoDB server.

If you are using Docker Swarm, you can instead set the following variables to enable Docker secrets:
* `MICA_ADMINISTRATOR_PASSWORD_FILE`
* `OPAL_ADMINISTRATOR_PASSWORD_FILE`
* `MONGO_INITDB_ROOT_PASSWORD_FILE`

**Volumes**  
Volumes can be mounted to different parts of the container:
* `/usr/share/mica2/forms` where you can place any study, population or DCE forms and search criteria you may want to add. The `forms` directory must have the following structure:
<pre>
    <b>forms</b>
    ├── <b>individual-studies</b>
    │   ├── data-collection-event.json
    │   ├── individual-study.json
    │   └── population.json
    └── <b>search-criteria</b>
        └── mica_search_criteria.json
</pre>
* `/usr/share/mica2/opal_creds` where you can place any Opal credentials (JSON format) you may want to add;
* `/usr/share/mica2/conf/taxonomies/mica-taxonomy.yml` where you can define custom taxonomies (e.g. taxonomies you have added to Opal) that can be used by the `mica-search-es` plugin to build queries;
* `/srv` where data that should be persisted is stored.

## Running

Create a `docker-compose.yml` file (examples bellow) and run `docker-compuse up`.

**Basic Example:**
```yml
version: '3.2'

services:
  mica:
    image: docker-registry.inesctec.pt/coral/coral-docker-images/coral-mica-docker:1.3.3
    container_name: mica
    environment:
    - MICA_ADMINISTRATOR_PASSWORD=<INSERT_VALUE_HERE>
    - MICA_ANONYMOUS_PASSWORD=<INSERT_VALUE_HERE>
    - OPAL_PORT_8443_TCP_ADDR=<INSERT_VALUE_HERE>
    - OPAL_PORT_8443_TCP_PORT=<INSERT_VALUE_HERE>
    - OPAL_ADMINISTRATOR_PASSWORD=<INSERT_VALUE_HERE>
    - AGATE_PORT_8444_TCP_ADDR=<INSERT_VALUE_HERE>
    - AGATE_PORT_8444_TCP_PORT=<INSERT_VALUE_HERE>
    - MONGO_PORT_27017_TCP_ADDR=<INSERT_VALUE_HERE>
    - MONGO_PORT_27017_TCP_PORT=<INSERT_VALUE_HERE>
    - MONGO_INITDB_ROOT_USERNAME=<INSERT_VALUE_HERE>
    - MONGO_INITDB_ROOT_PASSWORD=<INSERT_VALUE_HERE>
    volumes:
    - <PATH_TO_FORMS_DIR>:/usr/share/mica2/forms
    - <PATH_TO_OPAL_CREDS_DIR>:/usr/share/mica2/opal_creds
    - <PATH_TO_TAXONOMY_FILE>:/usr/share/mica2/conf/taxonomies/mica-taxonomy.yml
    - mica:/srv

volumes:
  mica:
```
**Docker Swarm Example:**
```yml
version: '3.2'

services:
  mica:
    image: docker-registry.inesctec.pt/coral/coral-docker-images/coral-mica-docker:1.3.3
    container_name: mica
    environment:
    - MICA_ADMINISTRATOR_PASSWORD_FILE=/run/secrets/MICA_ADMINISTRATOR_PASSWORD
    - MICA_ANONYMOUS_PASSWORD=<INSERT_VALUE_HERE>
    - OPAL_PORT_8443_TCP_ADDR=<INSERT_VALUE_HERE>
    - OPAL_PORT_8443_TCP_PORT=<INSERT_VALUE_HERE>
    - OPAL_ADMINISTRATOR_PASSWORD_FILE=/run/secrets/OPAL_ADMINISTRATOR_PASSWORD
    - AGATE_PORT_8444_TCP_ADDR=<INSERT_VALUE_HERE>
    - AGATE_PORT_8444_TCP_PORT=<INSERT_VALUE_HERE>
    - MONGO_PORT_27017_TCP_ADDR=<INSERT_VALUE_HERE>
    - MONGO_PORT_27017_TCP_PORT=<INSERT_VALUE_HERE>
    - MONGO_INITDB_ROOT_USERNAME=<INSERT_VALUE_HERE>
    - MONGO_INITDB_ROOT_PASSWORD_FILE=/run/secrets/MONGO_INITDB_ROOT_PASSWORD
    secrets:
    - MICA_ADMINISTRATOR_PASSWORD
    - OPAL_ADMINISTRATOR_PASSWORD
    - MONGO_INITDB_ROOT_PASSWORD
    volumes:
    - <PATH_TO_FORMS_DIR>:/usr/share/mica2/forms
    - <PATH_TO_OPAL_CREDS_DIR>:/usr/share/mica2/opal_creds
    - <PATH_TO_TAXONOMY_FILE>:/usr/share/mica2/conf/taxonomies/mica-taxonomy.yml
    - mica:/srv

secrets:
  MICA_ADMINISTRATOR_PASSWORD:
    external: true
  OPAL_ADMINISTRATOR_PASSWORD:
    external: true
  MONGO_INITDB_ROOT_PASSWORD:
    external: true

volumes:
  mica:
```

---
master: [![pipeline status](https://gitlab.inesctec.pt/coral/coral-docker-images/coral-mica-docker/badges/master/pipeline.svg)](https://gitlab.inesctec.pt/coral/coral-docker-images/coral-mica-docker/commits/master)

dev: &emsp;&ensp;[![pipeline status](https://gitlab.inesctec.pt/coral/coral-docker-images/coral-mica-docker/badges/dev/pipeline.svg)](https://gitlab.inesctec.pt/coral/coral-docker-images/coral-mica-docker/commits/dev)
