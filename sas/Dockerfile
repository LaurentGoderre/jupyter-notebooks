FROM jupyter/minimal-notebook:2bfbb7d17524
LABEL maintainer="William Hearn <william.hearn@canada.ca>"
LABEL site="https://jupyterhub.readthedocs.io/en/stable/"

ENV SASPY_VERSION="2.2.4"

USER root

# apt-get and system utilities
RUN apt-get update && apt-get install -y \
    curl apt-transport-https debconf-utils \
    && rm -rf /var/lib/apt/lists/*

RUN pip install sas_kernel \
                azure-storage \
                pandas && \
    mkdir /opt/sasinside/

COPY conf/sascfg.py /opt/conda/lib/python3.6/site-packages/saspy/sascfg.py
COPY conf/SASDeploymentManager/ /opt/sasinside/SASHome/SASDeploymentManager/

RUN chown -R $NB_USER:$NB_GID /opt/sasinside/

RUN jupyter nbextension install --py sas_kernel.showSASLog && \
    jupyter nbextension enable sas_kernel.showSASLog --py && \
    jupyter nbextension install --py sas_kernel.theme && \
    jupyter nbextension enable sas_kernel.theme --py && \
    jupyter nbextension list

# java 10 install
RUN apt-get update \
  && apt-get -y install software-properties-common python-software-properties \
  && add-apt-repository ppa:linuxuprising/java \
  && apt-get update && \
  echo oracle-java10-installer shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections \
  && apt-get -y install oracle-java10-installer \
  && apt-get -y remove software-properties-common \
  && apt-get clean autoclean -y \
  && apt-get autoremove -y \
  && rm -rf /var/lib/apt \
    /var/lib/dpkg/arch \
    /var/lib/dpkg/available \
    /var/lib/dpkg/cmethopt \
    /var/lib/dpkg/diversions \
    /var/lib/dpkg/diversions-old \
    /var/lib/dpkg/lock \
    /var/lib/dpkg/parts \
    /var/lib/dpkg/statoverride \
    /var/lib/dpkg/status-old \
    /var/lib/dpkg/triggers \
    /var/lib/cache \
    /var/lib/log

# Blobfuse
ENV BUILD_DEPS="uuid-dev \
                libgcrypt11-dev \
                golang-go \
                build-essential \
                zlib1g-dev \
                pkg-config \
                libfuse-dev \
                cmake"

RUN apt-get update \
    && apt-get install -y --no-install-recommends $BUILD_DEPS fuse \
                                                              gnutls-dev \
                                                              libgnutls30 \
                                                              libcurl3-gnutls \
                                                              libcurl4-openssl-dev \
    && git clone https://github.com/Azure/azure-storage-fuse \
    && cd azure-storage-fuse \
    && ./build.sh \
    && mv build/blobfuse /usr/local/bin \
    && cd .. \
    && rm -rf azure-storage-fuse/ \
    && apt-get purge -y --auto-remove $BUILD_DEPS \
    && rm -rf /var/lib/apt/lists/*

COPY scripts/mnt-blobfuse.sh /usr/local/bin/

COPY conf/docs/ /home/$NB_USER/docs/

RUN chown -R $NB_USER:$NB_GID /home/$NB_USER/docs/

# custom MS repository
RUN curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
RUN curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list > /etc/apt/sources.list.d/mssql-release.list

# SQL Server drivers
RUN  apt-get update && ACCEPT_EULA=Y apt-get install -y msodbcsql mssql-tools && \
    echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc && \
    /bin/bash -c "source ~/.bashrc"

USER $NB_USER
