FROM debian:jessie

MAINTAINER fourquant <info@4quant.com>

ENV DEBIAN_FRONTEND noninteractive
ENV CONDA_DIR /opt/conda

# Core installs
RUN apt-get update && \
    apt-get install -y git vim wget build-essential python-dev ca-certificates bzip2 libsm6 && \
    apt-get clean

# Install conda
RUN echo 'export PATH=$CONDA_DIR/bin:$PATH' > /etc/profile.d/conda.sh && \
    wget --quiet hhttps://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    /bin/bash /Miniconda3-latest-Linux-x86_64.sh -b -p $CONDA_DIR && \
    rm Miniconda3-latest-Linux-x86_64.sh && \
    $CONDA_DIR/bin/conda install --yes conda==4.2.9

# Create a user
RUN useradd -m -s /bin/bash fourquant
RUN chown -R fourquant:fourquant $CONDA_DIR

# Open port
EXPOSE 8888

# Env vars
USER fourquant
ENV HOME /home/fourquant
ENV SHELL /bin/bash
ENV USER fourquant
ENV PATH $CONDA_DIR/bin:$PATH
WORKDIR $HOME

# Setup ipython
RUN conda install --yes ipython-notebook terminado && conda clean -yt
RUN ipython profile create

# Switch to root for permissions
USER root

# Java setup
RUN apt-get install -y default-jre

# Spark setup 
## Spark dependencies
ENV APACHE_SPARK_VERSION 2.0.1
ENV PYJ_VERSION py4j-0.10.1-src.zip
RUN apt-get -y update && \
    apt-get install -y --no-install-recommends openjdk-7-jre-headless && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
RUN cd /tmp && \
        wget -q http://d3kbcqa49mib13.cloudfront.net/spark-${APACHE_SPARK_VERSION}-bin-hadoop2.6.tgz && \
        tar xzf spark-${APACHE_SPARK_VERSION}-bin-hadoop2.6.tgz -C /usr/local && \
        rm spark-${APACHE_SPARK_VERSION}-bin-hadoop2.6.tgz
RUN cd /usr/local && ln -s spark-${APACHE_SPARK_VERSION}-bin-hadoop2.6 spark

# Spark and Mesos config
ENV SPARK_HOME /usr/local/spark
ENV PATH $PATH:$SPARK_HOME/bin
RUN sed 's/log4j.rootCategory=INFO/log4j.rootCategory=ERROR/g' $SPARK_HOME/conf/log4j.properties.template > $SPARK_HOME/conf/log4j.properties
ENV _JAVA_OPTIONS "-Xms512m -Xmx4g" 
ENV PYTHONPATH $SPARK_HOME/python:$SPARK_HOME/python/lib/$PYJ_LIB_VERSION
ENV SPARK_OPTS --driver-java-options=-Xms1024M --driver-java-options=-Xmx4096M --driver-java-options=-Dlog4j.logLevel=info

# Install useful Python packages

RUN apt-get install -y libxrender1 fonts-dejavu && apt-get clean
RUN conda create --yes -q -n python3.4-env python=3.4 nose numpy pandas scikit-learn scikit-image matplotlib scipy seaborn sympy cython patsy statsmodels cloudpickle numba bokeh pillow ipython jsonschema boto
ENV PATH $CONDA_DIR/bin:$PATH
RUN conda install --yes numpy pandas scikit-learn scikit-image matplotlib scipy seaborn sympy cython patsy statsmodels cloudpickle numba bokeh pillow && conda clean -yt
RUN /bin/bash -c "pip install mistune"

# Bolt setup
RUN apt-get install -y git python-pip ipython gcc
RUN git clone https://github.com/bolt-project/bolt
RUN /bin/bash -c "pip install -r bolt/requirements.txt"
ENV BOLT_ROOT $HOME/bolt
ENV PATH $PATH:$BOLT_ROOT/bin
ENV PYTHONPATH $PYTHONPATH:$BOLT_ROOT

# additional libraries for Keras and Elephas
# RUN apt-get --no-install-recommends -y --force-yes install liblapack-dev libblas-dev gfortran


USER fourquant

# Install Python 3 Tensorflow
RUN conda install --quiet --yes 'tensorflow=0.9.0'
# Keras
RUN conda install --channel https://conda.anaconda.org/KEHANG --quiet --yes 'keras=1.0.8'
# Use the latest version of hyperopts (python 3.5 compatibility)
RUN pip install https://github.com/hyperopt/hyperopt/archive/master.zip
# Elephas for distributed spark
RUN pip install elephas

# Setup for standard runtime
# Add the notebooks directory
ADD notebooks $HOME/notebooks

# Set up the kernelspec
RUN /opt/conda/envs/python3.4-env/bin/ipython kernelspec install-self

# Set permissions on the notebooks
RUN chown -R fourquant:fourquant $HOME/notebooks

# Switch back to non-root user
USER fourquant

WORKDIR $HOME/notebooks

# Setup Spark + IPython env vars
ENV PYSPARK_PYTHON=/opt/conda/bin/python
ENV PYSPARK_DRIVER_PYTHON=/opt/conda/bin/python
ENV IPYTHON 1
ENV IPYTHON_OPTS "notebook --ip=0.0.0.0"

CMD /bin/bash -c pyspark
