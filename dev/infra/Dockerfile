#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# Image for building and testing Spark branches. Based on Ubuntu 20.04.
# See also in https://hub.docker.com/_/ubuntu
FROM ubuntu:focal-20221019

ENV FULL_REFRESH_DATE 20231117

ENV DEBIAN_FRONTEND noninteractive
ENV DEBCONF_NONINTERACTIVE_SEEN true

RUN apt-get update && apt-get install -y \
    build-essential \
    ca-certificates \
    curl \
    gfortran \
    git \
    gnupg \
    libcurl4-openssl-dev \
    libfontconfig1-dev \
    libfreetype6-dev \
    libfribidi-dev \
    libgit2-dev \
    libharfbuzz-dev \
    libjpeg-dev \
    liblapack-dev \
    libopenblas-dev \
    libpng-dev \
    libpython3-dev \
    libssl-dev \
    libtiff5-dev \
    libxml2-dev \
    nodejs \
    npm \
    openjdk-17-jdk-headless \
    pandoc \
    pkg-config \
    python3-pip \
    python3-setuptools \
    python3.8 \
    python3.9 \
    qpdf \
    r-base \
    ruby \
    ruby-dev \
    software-properties-common \
    wget \
    zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*


RUN echo 'deb https://cloud.r-project.org/bin/linux/ubuntu focal-cran40/' >> /etc/apt/sources.list
RUN gpg --keyserver hkps://keyserver.ubuntu.com --recv-key E298A3A825C0D65DFD57CBB651716619E084DAB9
RUN gpg -a --export E084DAB9 | apt-key add -
RUN add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu focal-cran40/'

RUN Rscript -e "install.packages(c('devtools', 'knitr', 'markdown',  \
    'rmarkdown', 'testthat', 'devtools', 'e1071', 'survival', 'arrow',  \
    'ggplot2', 'mvtnorm', 'statmod', 'xml2'), repos='https://cloud.r-project.org/')"

# See more in SPARK-39959, roxygen2 < 7.2.1
RUN Rscript -e "devtools::install_version('roxygen2', version='7.2.0', repos='https://cloud.r-project.org')"
RUN Rscript -e "devtools::install_version('lintr', version='2.0.1', repos='https://cloud.r-project.org')"
RUN Rscript -e "devtools::install_version('pkgdown', version='2.0.1', repos='https://cloud.r-project.org')"
RUN Rscript -e "devtools::install_version('preferably', version='0.4', repos='https://cloud.r-project.org')"

# See more in SPARK-39735
ENV R_LIBS_SITE "/usr/local/lib/R/site-library:${R_LIBS_SITE}:/usr/lib/R/library"


RUN curl -sS https://bootstrap.pypa.io/get-pip.py | python3.9


RUN add-apt-repository ppa:pypy/ppa
RUN mkdir -p /usr/local/pypy/pypy3.8 && \
    curl -sqL https://downloads.python.org/pypy/pypy3.8-v7.3.11-linux64.tar.bz2 | tar xjf - -C /usr/local/pypy/pypy3.8 --strip-components=1 && \
    ln -sf /usr/local/pypy/pypy3.8/bin/pypy /usr/local/bin/pypy3.8 && \
    ln -sf /usr/local/pypy/pypy3.8/bin/pypy /usr/local/bin/pypy3
RUN curl -sS https://bootstrap.pypa.io/get-pip.py | pypy3
RUN pypy3 -m pip install numpy 'six==1.16.0' 'pandas<=2.1.4' scipy coverage matplotlib


ARG BASIC_PIP_PKGS="numpy pyarrow>=14.0.0 six==1.16.0 pandas<=2.1.4 scipy unittest-xml-reporting plotly>=4.8 mlflow>=2.8.1 coverage matplotlib openpyxl memory-profiler>=0.61.0 scikit-learn>=1.3.2"
# Python deps for Spark Connect
ARG CONNECT_PIP_PKGS="grpcio==1.59.3 grpcio-status==1.59.3 protobuf==4.25.1 googleapis-common-protos==1.56.4"


RUN python3.9 -m pip install $BASIC_PIP_PKGS $CONNECT_PIP_PKGS
# Add torch as a testing dependency for TorchDistributor and DeepspeedTorchDistributor
RUN python3.9 -m pip install 'torch<=2.0.1' torchvision --index-url https://download.pytorch.org/whl/cpu
RUN python3.9 -m pip install deepspeed torcheval

# Install Python 3.10 at the last stage to avoid breaking Python 3.9
RUN add-apt-repository ppa:deadsnakes/ppa
RUN apt-get update && apt-get install -y \
    python3.10 python3.10-distutils \
    && rm -rf /var/lib/apt/lists/*
RUN curl -sS https://bootstrap.pypa.io/get-pip.py | python3.10
RUN python3.10 -m pip install $BASIC_PIP_PKGS $CONNECT_PIP_PKGS
RUN python3.10 -m pip install 'torch<=2.0.1' torchvision --index-url https://download.pytorch.org/whl/cpu
RUN python3.10 -m pip install deepspeed torcheval

# Install Python 3.11 at the last stage to avoid breaking the existing Python installations
RUN add-apt-repository ppa:deadsnakes/ppa
RUN apt-get update && apt-get install -y \
    python3.11 python3.11-distutils \
    && rm -rf /var/lib/apt/lists/*
RUN curl -sS https://bootstrap.pypa.io/get-pip.py | python3.11
RUN python3.11 -m pip install $BASIC_PIP_PKGS $CONNECT_PIP_PKGS
RUN python3.11 -m pip install 'torch<=2.0.1' torchvision --index-url https://download.pytorch.org/whl/cpu
RUN python3.11 -m pip install deepspeed torcheval

# Install Python 3.12 at the last stage to avoid breaking the existing Python installations
RUN add-apt-repository ppa:deadsnakes/ppa
RUN apt-get update && apt-get install -y \
    python3.12 python3.12-distutils \
    && rm -rf /var/lib/apt/lists/*
RUN curl -sS https://bootstrap.pypa.io/get-pip.py | python3.12
RUN python3.12 -m pip install $BASIC_PIP_PKGS $CONNECT_PIP_PKGS
# TODO(SPARK-46078) Use official one instead of nightly build when it's ready
RUN python3.12 -m pip install --pre torch --index-url https://download.pytorch.org/whl/nightly/cpu
RUN python3.12 -m pip install torchvision --index-url https://download.pytorch.org/whl/cpu
RUN python3.12 -m pip install torcheval
