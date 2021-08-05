FROM ubuntu:20.04
# SNAP is working with python 3.8
#LABEL authors="Carmen Tawalika,Markus Neteler"
#LABEL maintainer="tawalika@mundialis.de,neteler@mundialis.de"
LABEL authors="orttak"

ENV DEBIAN_FRONTEND noninteractive

USER root

# Install dependencies and tools
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends --no-install-suggests \
    build-essential \
    libgfortran5 \
    locales \
    python3 \
    python3-dev \
    python3-pip \
    python3-setuptools \
    libpq-dev \
    git \
    vim \
    wget \
    zip \
    && apt-get autoremove -y \
    && apt-get clean -y

# Set the locale
ENV LANG en_US.utf8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.utf8

# SNAP wants the current folder '.' included in LD_LIBRARY_PATH
ENV LD_LIBRARY_PATH ".:$LD_LIBRARY_PATH"

# install SNAPPY
RUN apt-get install default-jdk maven -y
ENV JAVA_HOME "/usr/lib/jvm/java-11-openjdk-amd64/"
RUN update-alternatives --install /usr/bin/python python /usr/bin/python3 1
COPY snap /src/snap
RUN bash /src/snap/install.sh
RUN update-alternatives --remove python /usr/bin/python3

# due to Ubuntu-GDAL being too old we prefer to use the SNAP-bundled GDAL:
# INFO: org.esa.s2tbx.dataio.gdal.GDALVersion: GDAL not found on system. Internal GDAL 3.0.0 from distribution will be used. (f1)

# path
RUN echo "export PATH=\$PATH:/usr/local/snap/bin/:/root/.snap/auxdata/gdal/gdal-3-0-0/bin" >> /root/.bashrc
#this gdal is not working so we install gdal in next session

# tests
# https://senbox.atlassian.net/wiki/spaces/SNAP/pages/50855941/Configure+Python+to+use+the+SNAP-Python+snappy+interface
RUN (cd /root/.snap/snap-python/snappy && python3 setup.py install) 
RUN /usr/bin/python3 -c 'from snappy import ProductIO'
RUN /usr/bin/python3 /src/snap/about.py
RUN apt-get install -y gdal-bin \
		       libgdal-dev

# When using SNAP from Python, either do: sys.path.append('/root/.snap/snap-python')

COPY requirements.txt ./
RUN python3 -m pip install -r requirements.txt
RUN cp -r /root/.snap/snap-python/snappy /usr/local/lib/python3.8/dist-packages

# Reduce the image size
RUN apt-get autoremove -y
RUN apt-get clean -y
RUN rm -rf /src


# Generate default config and disable authentication
RUN jupyter-notebook --generate-config --allow-root -y
#RUN sed -i "s/#c.NotebookApp.token = '<generated>'/c.NotebookApp.token = ''/" /root/.jupyter/jupyter_notebook_config.py
WORKDIR /home
RUN jupyter nbextension enable --py widgetsnbextension
CMD jupyter-lab --port=8888 --no-browser --ip=0.0.0.0 --allow-root