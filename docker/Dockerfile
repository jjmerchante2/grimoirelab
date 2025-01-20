# Copyright (C) 2016-2022 Bitergia
# GPLv3 License

FROM python:3.11-slim-bullseye

LABEL maintainer="Santiago Dueñas <sduenas@bitergia.com>"

ENV DEBIAN_FRONTEND noninteractive
ENV DEPLOY_USER grimoirelab
ENV DEPLOY_USER_DIR /home/${DEPLOY_USER}
ENV CONF_DIR ${DEPLOY_USER_DIR}/conf
ENV SCRIPTS_DIR ${DEPLOY_USER_DIR}/scripts
ENV GRIMOIRELAB_RELEASE "2.x"

# Create a user and a group
RUN groupadd -r $DEPLOY_USER && useradd -r -m -g $DEPLOY_USER $DEPLOY_USER

# Install dependencies
RUN apt-get update && \
    apt-get -y install --no-install-recommends \
        bash locales \
        gcc \
        git git-core \
        mariadb-client \
        libmariadbclient-dev-compat \
        unzip curl wget sudo ssh \
        && \
    apt-get clean && \
    find /var/lib/apt/lists -type f -delete

RUN echo "${DEPLOY_USER} ALL=NOPASSWD: ALL" >> /etc/sudoers

RUN mkdir -p ${DEPLOY_USER_DIR}/logs ; chown -R ${DEPLOY_USER}:${DEPLOY_USER} ${DEPLOY_USER_DIR}/logs
VOLUME ["${DEPLOY_USER_DIR}/logs"]

RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    echo 'LANG="en_US.UTF-8"'>/etc/default/locale && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=en_US.UTF-8

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8
ENV LANG C.UTF-8

USER ${DEPLOY_USER}

# Install GrimoireLab and dependencies
#
# We use a virtual environment because it makes easier
# to find the exact location of the packages installed
# and where the static files are available.

# Install Poetry
RUN curl -sSL https://install.python-poetry.org | python3 -
ENV PATH="$DEPLOY_USER_DIR/.local/bin:$PATH"

ENV \
    # Disable some unused uWSGI features, saving dependencies
    # Thanks to https://stackoverflow.com/a/25260588/90297
    UWSGI_PROFILE_OVERRIDE=ssl=false;xml=false;routing=false

WORKDIR ${DEPLOY_USER_DIR}
RUN git clone -b master --depth 1 https://github.com/jjmerchante2/grimoirelab.git
ADD ./docker/grimoirelab_core-0.0.1rc2-py3-none-any.whl ${DEPLOY_USER_DIR}/grimoirelab

WORKDIR ${DEPLOY_USER_DIR}/grimoirelab

USER root
RUN python -m venv /opt/venv/ && \
    . /opt/venv/bin/activate && \
    poetry install && \
    pip install --force-reinstall grimoirelab_core-0.0.1rc2-py3-none-any.whl
RUN chown -R $DEPLOY_USER:$DEPLOY_USER /opt/venv

COPY ./docker/entrypoint.sh /usr/local/bin/
RUN chmod 755 /usr/local/bin/entrypoint.sh

USER ${DEPLOY_USER}

ENTRYPOINT ["entrypoint.sh"]
CMD ["grimoirelab"]
