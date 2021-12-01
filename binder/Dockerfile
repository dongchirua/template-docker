FROM continuumio/miniconda3

LABEL repo2docker.ref="master"
LABEL repo2docker.repo="https://github.com/dongchirua/template-docker"
LABEL repo2docker.version="2021.03.0+70.g43891a6"
LABEL tljh_repo2docker.mem_limit="1G"
LABEL tljh_repo2docker.cpu_limit="1"
LABEL tljh_repo2docker.display_name="quick-start"
LABEL tljh_repo2docker.image_name="quick-start:master"

ARG NB_USER
ARG NB_UID
EXPOSE 8888
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get -qq update && \
    apt-get -qq install --yes --no-install-recommends locales build-essential gnupg2
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && locale-gen
ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV SHELL /bin/bash
ENV USER ${NB_USER}
ENV HOME /home/${NB_USER}
RUN groupadd --gid ${NB_UID} ${NB_USER} && \ 
    useradd --comment "Default user" \ 
            --create-home --gid ${NB_UID} \
            --no-log-init --shell /bin/bash \ 
            --uid ${NB_UID} ${NB_USER}
RUN wget --quiet -O - https://deb.nodesource.com/gpgkey/nodesource.gpg.key |  apt-key add - && \
    DISTRO="bionic" && \
    echo "deb https://deb.nodesource.com/node_16.x $DISTRO main" >> /etc/apt/sources.list.d/nodesource.list && \ 
    echo "deb-src https://deb.nodesource.com/node_16.x $DISTRO main" >> /etc/apt/sources.list.d/nodesource.list
RUN apt-get -qq update && \
    apt-get -qq install --yes --no-install-recommends less nodejs unzip > /dev/null
ENV APP_BASE /srv
ENV NPM_DIR ${APP_BASE}/npm
ENV NPM_CONFIG_GLOBALCONFIG ${NPM_DIR}/npmrc
ENV CONDA_DIR ${APP_BASE}/conda
ENV NB_PYTHON_PREFIX ${CONDA_DIR}/envs/notebook
ENV NB_ENVIRONMENT_FILE /tmp/env/base.yml
ENV MINICONDA /opt/conda
ENV KERNEL_PYTHON_PREFIX ${NB_PYTHON_PREFIX}
ENV PATH ${MINICONDA}/bin:${NB_PYTHON_PREFIX}/bin:${CONDA_DIR}/bin:${NPM_DIR}/bin:${PATH}
COPY --chown=${NB_USER}:${NB_UID}  binder/activate-conda.sh  /etc/profile.d/activate-conda.sh
RUN mkdir -p ${NPM_DIR} && chown -R ${NB_USER}:${NB_USER} ${NPM_DIR}
USER root
ARG REPO_DIR=${HOME}
ENV REPO_DIR ${REPO_DIR}
WORKDIR ${REPO_DIR}
RUN chown ${NB_USER}:${NB_USER} ${REPO_DIR}
ENV PATH ${HOME}/.local/bin:${REPO_DIR}/.local/bin:${PATH}
ENV CONDA_DEFAULT_ENV ${KERNEL_PYTHON_PREFIX}
COPY binder/apt.txt ${REPO_DIR}/binder/apt.txt
RUN cat ${REPO_DIR}/binder/apt.txt | while read line; do apt-get install --yes --no-install-recommends $line; done
COPY --chown=${NB_USER}:${NB_UID} binder/environment.yml ${REPO_DIR}/binder/environment.yml
RUN conda install mamba -n base -c conda-forge
COPY binder/base.yml $NB_ENVIRONMENT_FILE
RUN TIMEFORMAT='time: %3R' bash -c 'time mamba env create -p ${NB_PYTHON_PREFIX} -f ${NB_ENVIRONMENT_FILE} && time mamba clean --all -f -y && mamba list && time conda clean -a --force-pkgs-dirs -y'
RUN ${NB_PYTHON_PREFIX}/bin/npm config --global set prefix ${NPM_DIR}
RUN conda config --system --set auto_update_conda false && \
    conda config --system --set show_channel_urls true && \
    conda config --system --set channel_priority "flexible"
RUN npm config --global set prefix ${NPM_DIR}
RUN chown -R ${NB_USER}:${NB_UID} ${REPO_DIR}
RUN chown -R ${NB_USER}:${NB_UID} ${APP_BASE}
RUN chown -R ${NB_USER}:${NB_UID} ${MINICONDA}
RUN chown -R ${NB_USER}:${NB_UID} /usr/local/share
RUN apt-get -qq purge && apt-get -qq clean && rm -rf /var/lib/apt/lists/*
USER ${NB_USER}
RUN TIMEFORMAT='time: %3R' bash -c 'time mamba env update -p ${NB_PYTHON_PREFIX} -f "binder/environment.yml" && time mamba clean --all -f -y && mamba list -p ${NB_PYTHON_PREFIX} '
COPY --chown=${NB_USER}:${NB_UID} . ${REPO_DIR}
RUN conda init bash
USER root
RUN chmod +x binder/postBuild
USER ${NB_USER}
RUN ./binder/postBuild
ENV PYTHONUNBUFFERED=1
USER root
COPY binder/python3-login /usr/local/share/python3-login
COPY binder/repo2docker-entrypoint /usr/local/share/repo2docker-entrypoint
RUN chmod +x /usr/local/share/python3-login
RUN chmod +x /usr/local/share/repo2docker-entrypoint
USER ${NB_USER}
ENTRYPOINT ["/usr/local/share/repo2docker-entrypoint"]
CMD ["jupyter", "notebook", "--ip", "0.0.0.0", "--port", "8888"]
