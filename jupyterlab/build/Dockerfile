# This file creates a container that runs a jupyter lab server on Raspberry Pi
#
# Originally from: https://github.com/jiwidi/jupyter-lab-docker-rpi/blob/master/Dockerfile

FROM balenalib/raspberry-pi-python:3.7-20200518

ENV NODE_OPTIONS=--max-old-space-size=2048

WORKDIR /root

# Update pip and install jupyter
RUN apt-get update && apt-get upgrade
RUN apt-get install -y apt-utils wget git curl
RUN apt-get install -y libncurses5-dev 
RUN apt-get install -y libzmq-dev 
RUN apt-get install -y libfreetype6-dev 
RUN apt-get install -y libpng-dev
RUN apt-get install -y python-dev --fix-missing 
RUN apt-get install -y patch
RUN apt-get install -y build-essential
RUN apt-get install -y libpq-dev 
RUN apt-get install -y libstdc++6
RUN apt-get install -y libatlas3-base 
RUN apt-get install -y libgfortran5
RUN apt-get install -y libtinfo6
RUN apt-get install -y libncurses6

RUN pip3 install --upgrade pip -i https://www.piwheels.org/simple
RUN pip3 install cython -i https://www.piwheels.org/simple
RUN pip3 install gnureadline ipywidgets jupyter jupyterlab==2.0.0 -i https://www.piwheels.org/simple

RUN wget -O - https://deb.nodesource.com/setup_12.x | bash
RUN apt-get install -y nodejs g++ make software-properties-common libzmq3-dev

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update && apt-get install yarn

# Configure jupyter
RUN jupyter nbextension enable --py widgetsnbextension
RUN jupyter serverextension enable --py jupyterlab
RUN jupyter notebook --generate-config
RUN mkdir notebooks
#RUN sed -i "/c.NotebookApp.open_browser/c c.NotebookApp.open_browser = False" /root/.jupyter/jupyter_notebook_config.py \
#        && sed -i "/c.NotebookApp.ip/c c.NotebookApp.ip = '*'" /root/.jupyter/jupyter_notebook_config.py \
#        && sed -i "/c.NotebookApp.notebook_dir/c c.NotebookApp.notebook_dir = '/root'" /root/.jupyter/jupyter_notebook_config.py \
#        && sed -i "/c.NotebookApp.password/c c.NotebookApp.password = 'sha1:5815fb7ca805:f09ed218dfcc908acb3e29c3b697079fea37486a'" /root/.jupyter/jupyter_notebook_config.py

VOLUME /root/notebooks

# Add Tini. Tini operates as a process subreaper for jupyter. This prevents kernel crashes.
ENV TINI_VERSION 0.18.0
ENV CFLAGS="-DPR_SET_CHILD_SUBREAPER=36 -DPR_GET_CHILD_SUBREAPER=37"

ADD https://github.com/krallin/tini/archive/v${TINI_VERSION}.tar.gz /root/v${TINI_VERSION}.tar.gz
RUN apt-get install -y cmake
RUN tar zxvf v${TINI_VERSION}.tar.gz \
        && cd tini-${TINI_VERSION} \
        && cmake . \
        && make \
        && cp tini /usr/bin/. \
        && cd .. \
        && rm -rf "./tini-${TINI_VERSION}" \
        && rm "./v${TINI_VERSION}.tar.gz"

ENTRYPOINT ["/usr/bin/tini", "--"]

EXPOSE 8888

COPY pip.conf /etc/pip.conf
RUN apt autoremove
WORKDIR /root/notebooks

ENV SHELL=/bin/bash

CMD ["jupyter", "lab", "--allow-root", "--no-browser"]
