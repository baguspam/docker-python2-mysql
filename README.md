# Python and MySQL for Bitbucket Pipelines

This repository contains a Dockerfile as well as a simple example that shows how you can run your own Docker container with Python and MySQL on Bitbucket Pipelines.

The Docker image is using Python 2.7.13 and MySQL 5.5

## Quickstart

### Using the image with Bitbucket Pipelines

Just copy/paste the YML below in your bitbucket-pipelines.yml and adapt the script to your needs.

```yaml
# This is a sample build configuration for Python.
# Only use spaces to indent your .yml configuration.
# -----
# You can specify a custom docker image from Dockerhub as your build environment.
image: mikemanger/python27-mysql

pipelines:
  default:
    - step:
        script: # Modify the commands below to build your repository.
          - /etc/init.d/mysql start
          - mysql -uroot -e 'create database python'
          - python test.py
```

### Using this in a script

You'll find a sample script in this repository in test.py. It simply connects to MySQL and then lists the existing databases.

```python
#!/usr/bin/python

import pymysql

# Open database connection
db = pymysql.connect("localhost","root" )

cur = db.cursor()

cur.execute("SHOW DATABASES")

# print all the first cell of all the rows
for row in cur.fetchall():
    print(row[0])

db.close()
```

## Create your own image

If you want to use a different version of Python you can simply create your own image for it. Just copy the content of the Dockerfile and replace the first line.

This image is built from the official Python image at https://hub.docker.com/_/python/ and you can find there all the different versions that are supported.

Your Dockerfile won't need to have an ENTRYPOINT or CMD line as Bitbucket Pipelines will run the script commands that you put in your bitbucket-pipelines.yml file instead.

```
FROM python:2.7.13
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update \
  && apt-get install -y mysql-server --no-install-recommends \
  && apt-get clean \
  && pip install PyMySQL \
  && pip install MySQL-python \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# This Dockerfile doesn't need to have an entrypoint and a command
# as Bitbucket Pipelines will overwrite it with a bash script.
```

### Build the image

```bash
docker build -t <your-docker-account>/python27-mysql .
```

### Run the image locally with bash to make some tests

```bash
docker run -i -t <your-docker-account>/python27-mysql /bin/bash
```

### Push the image back to the Docker Hub

```bash
docker push <your-docker-account>/python27-mysql
```