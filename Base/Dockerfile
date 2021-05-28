FROM ubuntu:focal-20210401

LABEL \
    "com.trilogy.company"="aurea" \
    "com.trilogy.team"="eng.qa.integration" \
    "com.trilogy.product"="generic" \
    "com.trilogy.service"="jenkins-agent" \
    "com.trilogy.stage"="dev" \
    "com.trilogy.maintainer.skype"="asuwalka" \
    "com.trilogy.maintainer.email"="ajay.suwalka@aurea.com"

# No interactive frontend during docker build
ENV DEBIAN_FRONTEND=noninteractive \
    DEBCONF_NONINTERACTIVE_SEEN=true

USER root

WORKDIR /home/ubuntuadmin

ENV JENKINS_HOME /home/ubuntuadmin
ENV _JAVA_OPTS -Dcom.sun.security.enableAIAcaIssuers=true

RUN apt-get -yqq update
RUN apt-get install -y --fix-missing locales && locale-gen en_US.UTF-8

RUN apt-get -yqq update && \
    apt-get install -yqq gnupg2 wget openssh-server imagemagick git \
    freetds-dev freetds-bin unixodbc-dev tdsodbc gcc libpq-dev python3-dev build-essential libaio-dev libssl-dev software-properties-common unzip openjdk-8-jdk

RUN wget -q https://www.browserstack.com/browserstack-local/BrowserStackLocal-linux-x64.zip && \
    unzip BrowserStackLocal-linux-x64.zip && \
    rm BrowserStackLocal-linux-x64.zip && \
    chmod +x BrowserStackLocal

RUN wget -q https://archive.apache.org/dist/tinkerpop/3.3.4/apache-tinkerpop-gremlin-console-3.3.4-bin.zip && \
    unzip apache-tinkerpop-gremlin-console-3.3.4-bin.zip -d /home && \
    rm apache-tinkerpop-gremlin-console-3.3.4-bin.zip && \
    mv /home/apache-tinkerpop-gremlin-console-3.3.4 /home/gremlin

COPY neptune-remote.yaml /home/gremlin/conf/

RUN echo 'alias gremlin="/home/gremlin/bin/gremlin.sh"' >> ~/.bashrc

RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
  && sh -c 'echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list' \
  && apt-get update \
  && apt-get install -yqq python3-pip python3-dev firefox tzdata google-chrome-stable \
  && cd /usr/local/bin \
  && ln -s /usr/bin/python3 python \
  && pip3 install --upgrade pip

# Install awscliv2
RUN wget -q "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -O "awscliv2.zip" \
&& unzip awscliv2.zip \
&& mkdir /usr/local/aws-cli-v2 \
&& ./aws/install -i /usr/local/aws-cli-v2 -b /usr/local/bin \
&& ln -s /usr/local/bin/aws /usr/local/bin/aws2

# Install oracle dpi
RUN wget -q "https://download.oracle.com/otn_software/linux/instantclient/19800/instantclient-basic-linux.x64-19.8.0.0.0dbru.zip" -O "instantclient-basic-linux.zip" \
&& wget -q "https://download.oracle.com/otn_software/linux/instantclient/19800/instantclient-sdk-linux.x64-19.8.0.0.0dbru.zip" -O "instantclient-sdk-linux.zip" \
&& unzip instantclient-basic-linux.zip \
&& unzip instantclient-sdk-linux.zip \
&& printf "%s/instantclient_19_8/libclntsh.so\n" "$PWD" >> /etc/ld.so.conf.d/oracle.conf

RUN wget -q "https://repo.maven.apache.org/maven2/io/qameta/allure/allure-commandline/2.12.0/allure-commandline-2.12.0.tgz" -O "allure-commandline-2.12.0.tgz" && \
    tar -zxvf allure-commandline-2.12.0.tgz -C /opt/ && ln -s /opt/allure-2.12.0/bin/allure /usr/bin/allure

ENV LANG='en_US.UTF-8' LANGUAGE='en_US:en' LC_ALL='en_US.UTF-8' PORT='8080' ORACLE_HOME='/usr/src/app/instantclient_19_8'
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME

RUN ldconfig

ENV PATH=$HOME/.local/bin:$PATH
# EXECUTOR_SUFFIX should be -dev on dev env and empty on prod
ENV EXECUTOR_SUFFIX=EXECUTOR_ENV_SUFFIX

RUN printf "[FreeTDS]\nDescription=FreeTDS Driver\nDriver=/usr/lib/x86_64-linux-gnu/odbc/libtdsodbc.so\nSetup=/usr/lib/x86_64-linux-gnu/odbc/libtdsS.so" > /etc/odbcinst.ini

# Install tesseract-ocr
ENV TESSDATA_PREFIX=/usr/local/share/tessdata

RUN add-apt-repository ppa:alex-p/tesseract-ocr && \
    apt-get install tesseract-ocr -y && \
    wget https://github.com/tesseract-ocr/tessdata/raw/master/eng.traineddata && \
    mkdir -p /usr/local/share/tessdata && \
    mv -v eng.traineddata /usr/local/share/tessdata

RUN apt-get install -yqq zip jq

RUN pip3 install e2e-pca-jsonpath-ng==1.5.2b20200928162102 --extra-index-url https://nexus.devfactory.com/repository/eng-pca-qe-pypi-release/simple/
RUN pip3 install eng.pca-automation-e2e --extra-index-url https://nexus.devfactory.com/repository/eng-pca-qe-pypi-release/simple/ --trusted-host pypi.org --trusted-host nexus.devfactory.com

# to support copy&paste
RUN apt-get update && apt-get install -y xclip xvfb
ENV DISPLAY=:99

# Create Jenkins User
RUN useradd jenkins -m -s /bin/bash

RUN usermod -a -G root jenkins

USER jenkins
