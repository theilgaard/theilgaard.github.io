---
layout: post
title: "A Dockerfile for Django in production"
date: 2019-10-29 +0100
categories: work
---
As python is becoming more and more popular, so is it's ecosystem and frameworks. Django is a popular web application framework for python, which has a great MVC architecture, comes with existing user management and a ton of plugins. 

As with almost all modern application development, we will have dependencies on other libraries. This will significantly speed up development time. However doing dependency management and environment isolation is always something to take into consideration when deploying to production.

Docker has become mainstream in production environments around the world, typically orchestrated by Docker Swarm or Kubernetes, and provides a level of isolation without spinning up a resource hungry virtual machine.

Recently I am developing a Django application I would like to deploy with Docker, and make as production ready as possible. Here is the Dockerfile I came up with.

{% highlight docker %}
FROM debian:buster

ENV POETRY_VERSION=0.12.17 \
    DOCKER_CONTAINER=1 \
    APPLICATION_CODE_FOLDER=/opt/app \
    PYTHONFAULTHANDLER=1 \
    PYTHONUNBUFFERED=1 \
    DJANGO_ENV=prod \
    DJANGO_SECRET_KEY=DEVELOPMENT_KEY

SHELL ["/bin/bash", "-c"]
WORKDIR ${APPLICATION_CODE_FOLDER}

# System dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    python3 \
    python3-dev \
    python3-pip \
    python3-venv && \
    python3 -m venv .venv 

# Copy in poetry and dependency list
COPY ./install/get-poetry.py ${APPLICATION_CODE_FOLDER}/install/get-poetry.py
COPY poetry.lock ${APPLICATION_CODE_FOLDER}/poetry.lock
COPY pyproject.toml ${APPLICATION_CODE_FOLDER}/pyproject.toml

# Create virtual environment and install dependencies
RUN source .venv/bin/activate && \
    python ./install/get-poetry.py -y && \
    source $HOME/.poetry/env && \
    poetry install --no-dev

# Copy in application code and collect static files
COPY . ${APPLICATION_CODE_FOLDER}
ENV PATH=${APPLICATION_CODE_FOLDER}/.venv/bin:$PATH
RUN python manage.py collectstatic --noinput

EXPOSE 8000

CMD ["uwsgi", "--ini", "/opt/app/uwsgi.ini"]
{% endhighlight %}

### Walkthrough 

#### Base image
The base image is debian which is intentionally fixed for version buster (10), this creates stability and control for the developer. The debian buster was released in 2019 and has [3 years support and 2 years LTS support according to documentation](https://www.debian.org/releases/).

I then proceed to define variables, that may be overwritten by the environment. Here an option will be added later, when doing automatic testing to run tests instead of the uWSGI webserver.

#### Dependency management
A layer is added with system dependencies. This allows for caching of the system dependency layer when updating the following layers. The logic behind this is that during development, the dependency list managed by poetry will likely change more rapidly, and thus require rebuilding, while the system dependency might change more seldom.

Next, poetry installation files, and the dependency list is copied in seperately. While copying everything in one layer would simplify the Dockerfile, this significantly slows the build as dependencies would not be cached in a layer by themselves. Thus requiring updating and download for all dependencies, for each code change.

#### Application code and entrypoint
Finally the application code is copied in, the `PATH` variable is updated to utilize the virtual environment, and the WSGI port is exposed at port 8000.

In this current image, only a running production uWSGI instance is considered. But the image could easily be updated to provide automatic testing in a continues integration environment, using the same dependency acquisition chain.