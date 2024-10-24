# ref: https://github.com/mamba-org/micromamba-docker/blob/main/Dockerfile

FROM condaforge/mambaforge:latest

LABEL maintainer="Ivan Ogasawara <ivan.ogasawara@gmail.com>"
LABEL org.opencontainers.image.title="Mhai"
LABEL org.opencontainers.image.authors="Mhai Team"
LABEL org.opencontainers.image.source="https://github.com/mentalhealthai-org/mhai-web"
LABEL org.opencontainers.image.version="latest"
LABEL org.opencontainers.image.description="Mhai Web"
LABEL org.mentalhealthai_org.config.version="latest"

# it is the default, but using it here to have it explicitly
USER root

SHELL ["/bin/bash", "-c"]
# Use bash in Dockerfile RUN commands and make sure bashrc is sourced when
# executing commands with /bin/bash -c
# Needed to have the micromamba activate command configured etc.

ENV ENV_NAME=mhai-web
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC
ARG UID=1000
ARG GID=1000

RUN apt-get update -y \
  && apt-get install -y \
    apt-utils \
    build-essential \
    curl \
    tini \
    nginx \
    snapd \
    sudo \
    tzdata \
    gcc-multilib \
    g++-multilib \
    openssl \
  && rm -rf /var/lib/apt/lists/* \
    /var/cache/apt/archives \
    /tmp/*

ENV USERNAME=mhai

RUN addgroup --gid ${GID} ${USERNAME} \
  && useradd --uid ${UID} --gid ${GID} -ms /bin/bash ${USERNAME} \
  && mkdir -p /opt/services/mhai-web \
  && chmod -R a+rwx /opt/conda /opt/services \
  && export ENV_NAME="$ENV_NAME" \
  && chown ${USERNAME}:${USERNAME} /opt/services \
  && echo "${USERNAME} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/${USERNAME} \
  && chmod 0440 /etc/sudoers.d/${USERNAME}

USER ${USERNAME}

WORKDIR /opt/services/mhai-web

COPY --chown=${USERNAME}:${USERNAME} ./conda/ /tmp/conda
# COPY --chown=${USERNAME}:${USERNAME} ./.github/workflows/.condarc /home/${USERNAME}/.condarc

ARG HTTP_PROXY
ARG HTTPS_PROXY

RUN mamba env create -n $ENV_NAME --file /tmp/conda/dev.yaml \
  && conda clean --all \
  && find /opt/conda/ -type f,l -name '*.pyc' -delete \
  && find /opt/conda/ -type f,l -name '*.js.map' -delete \
  && rm -rf /opt/conda/pkgs /tmp/*

ENV CONDA_PREFIX /opt/conda/envs/$ENV_NAME
ENV PATH ${CONDA_PREFIX}/bin:$PATH

# install dependencies
COPY --chown=${USERNAME}:${USERNAME} pyproject.toml poetry.lock /opt/services/mhai-web/
COPY --chown=${USERNAME}:${USERNAME} containers/django/scripts/install-deps.sh /opt/install-deps.sh
COPY --chown=${USERNAME}:${USERNAME} containers/django/scripts/activate.sh /opt/activate.sh

ARG ENV=dev

RUN conda init bash \
  && source ~/.bashrc \
  && source /opt/activate.sh \
  && conda env list \
  && /opt/install-deps.sh

COPY --chown=mhai-web:mhai-web ./containers/django/scripts/entrypoint.sh /opt/entrypoint.sh
RUN sed -i 's/\r$//g' /opt/entrypoint.sh
RUN chmod +x /opt/entrypoint.sh

ENTRYPOINT ["tini", "--", "/opt/entrypoint.sh"]
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
