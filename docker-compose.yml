# syntax = docker/dockerfile:experimental
# primo stadio: compilazione delle wheel
ARG PY_VER
FROM python:${PY_VER}-slim AS build

RUN apt-get update && apt-get install -y build-essential libpcre3 libpcre3-dev
RUN pip wheel --no-cache-dir --wheel-dir /wheels uwsgi


ARG PY_VER
FROM python:${PY_VER}-slim
COPY --from=build /wheels /wheels
RUN pip install --no-cache /wheels/*

# RUN --mount=type=cache,target=/root/.cache/pip pip install pyyaml
# Create a group and user to run our app
ARG APP_USER=www-data
#RUN groupadd -r ${APP_USER} && useradd --no-log-init -mr -g ${APP_USER} ${APP_USER}

# Install packages needed to run your application (not build deps):
#   mime-support -- for mime types when serving static files
#   postgresql-client -- for running database commands
# We need to recreate the /usr/share/man/man{1..8} directories first because
# they were clobbered by a parent image.
# ipython: a home is needed to store the history
RUN set -ex \
  && RUN_DEPS=" \
  mime-support \
  postgresql-client \
  libcap2 \
  libpcre3 \
  procps acl \
  cron \
  git \
  joe \
  gettext \
  curl \
  " \
  && seq 1 8 | xargs -I{} mkdir -p /usr/share/man/man{} \
  && apt-get update && apt-get install -y --no-install-recommends $RUN_DEPS $BUILD_DEPS \
  && mkdir -p /var/www/.ipython \
  && mkdir -p /var/log/uwsgi /var/log/django \
  && pip install --no-cache-dir --only-binary :all: --extra-index-url https://pypi.thux.dev/simple/ -U pip>=23.0 setuptools wheel uwsgi \
  && chown -R ${APP_USER} /var/www \
  && echo "alias ll='ls -l --color=auto'" > /var/www/.bashrc \
  && echo "alias ll='ls -l --color=auto'" > /root/.bashrc \
  && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false  \
  && rm -rf /var/lib/apt/lists/*
ARG POETRY_VERSION
#ENV POETRY_VERSION=${POETRY_VERSION:-1.1.14}
RUN echo POETRY_VERSION_=$POETRY_VERSION
RUN curl -sSL https://install.python-poetry.org | python3 - --version $POETRY_VERSION \
  && ln -s /root/.local/bin/poetry /usr/local/bin

VOLUME /var/log/django

VOLUME /var/log/uwsgi
