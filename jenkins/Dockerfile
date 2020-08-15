# https://www.jenkins.io/doc/tutorials/build-a-python-app-with-pyinstaller/
# docker container run --name jenkins-tutorial --rm --detach \
#   --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
#   --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
#   --volume jenkins-data:/var/jenkins_home \ 
#   --volume jenkins-docker-certs:/certs/client:ro \
#   --volume "$HOME":/home \ 
#   --publish 8080:8080 jenkinsci/blueocean

# Also refer https://hub.docker.com/layers/jenkinsci/blueocean/latest

FROM jenkinsci/blueocean:latest

LABEL Prabhakaran Sampath <prabhakaran@zealtechlab.co.in>

USER root

RUN apk add --no-cache curl tar bash procps git openssh less ca-certificates nodejs npm 

# Downloading and installing Maven
ARG MAVEN_VERSION=3.6.3
ARG USER_HOME_DIR="/root"
ARG SHA=c35a1803a6e70a126e80b2b3ae33eed961f83ed74d18fcd16909b2d44d7dada3203f1ffe726c17ef8dcca2dcaa9fca676987befeadc9b9f759967a8cb77181c0
ARG BASE_URL=https://mirrors.estointernet.in/apache/maven/maven-3/${MAVEN_VERSION}/binaries

RUN mkdir -p /usr/share/maven /usr/share/maven/ref \
  && echo "Downlaoding maven" \
  && curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
  \
  && echo "Checking download hash" \
  && echo "${SHA}  /tmp/apache-maven.tar.gz" | sha512sum -c - \
  \
  && echo "Unziping maven" \
  && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
  \
  && echo "Cleaning and setting links" \
  && rm -f /tmp/apache-maven.tar.gz \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"

# Downloading and installing Gradle
# 1- Define a constant with the version of gradle you want to install
ARG GRADLE_VERSION=6.5.1

# 2- Define the URL where gradle can be downloaded from
ARG GRADLE_BASE_URL=https://services.gradle.org/distributions

# 3- Define the SHA key to validate the gradle download
#    obtained from here https://gradle.org/release-checksums/
ARG GRADLE_SHA=50a7d30529fa939721fe9268a0205142f3f2302bcac5fb45b27a3902e58db54a

# 4- Create the directories, download gradle, validate the download, install it, remove downloaded file and set links
RUN mkdir -p /usr/share/gradle /usr/share/gradle/ref \
  && echo "Downlaoding gradle hash" \
  && curl -fsSL -o /tmp/gradle.zip ${GRADLE_BASE_URL}/gradle-${GRADLE_VERSION}-bin.zip \
  \
  && echo "Checking download hash" \
  && echo "${GRADLE_SHA}  /tmp/gradle.zip" | sha256sum -c - \
  \
  && echo "Unziping gradle" \
  && unzip -d /usr/share/gradle /tmp/gradle.zip \
   \
  && echo "Cleaning and setting links" \
  && rm -f /tmp/gradle.zip \
  && ln -s /usr/share/gradle/gradle-${GRADLE_VERSION} /usr/bin/gradle

# 5- Define environmental variables required by gradle
ENV GRADLE_VERSION 6.5.1
ENV GRADLE_HOME /usr/bin/gradle
ENV GRADLE_USER_HOME /cache

ENV PATH $PATH:$GRADLE_HOME/bin

VOLUME $GRADLE_USER_HOME

# CMD [""]

# Ansible - https://github.com/ansible/ansible/blob/stable-2.9/changelogs/CHANGELOG-v2.9.rst
ENV ANSIBLE_VERSION=2.9.11

RUN set -xe \
    && echo "****** Install system dependencies ******" \
    && apk add --no-cache --progress python3 openssl \
		ca-certificates git openssh sshpass \
	&& apk --update add --virtual build-dependencies \
		python3-dev libffi-dev openssl-dev build-base \
	\
	&& echo "****** Install ansible and python dependencies ******" \
    && pip3 install --upgrade pip \
	&& pip3 install ansible==${ANSIBLE_VERSION} boto3 \
    \
    && echo "****** Remove unused system librabies ******" \
	&& apk del build-dependencies \
	&& rm -rf /var/cache/apk/* 

RUN set -xe \
    && mkdir -p /etc/ansible \
    && echo -e "[local]\nlocalhost ansible_connection=local" > \
        /etc/ansible/hosts

# CMD ["ansible", "--version"]

#sonar Scanner - https://gist.github.com/henrist/302eeb29233ed45e8d2a119386fd394d
RUN set -eux

ARG SONAR_SCANNER_VERSION=4.4.0.2170

RUN mkdir -p /opt \
  && curl -fSL https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-${SONAR_SCANNER_VERSION}.zip -o /opt/sonar-scanner.zip \
  && unzip /opt/sonar-scanner.zip -d /opt \
  && rm /opt/sonar-scanner.zip \
  && ln -s /opt/sonar-scanner-${SONAR_SCANNER_VERSION}/bin/sonar-scanner /usr/bin/sonar-scanner

# installing specific list of plugins. see: https://github.com/jenkinsci/docker#preinstalling-plugins
COPY plugins.txt /usr/share/jenkins/plugins.txt
RUN install-plugins.sh < /usr/share/jenkins/plugins.txt

# ARG REF=/usr/share/jenkins/ref was already set in the base image
# Adding default Jenkins Seed Job
COPY jobs ${REF}/jobs/

# Selenium side runner local to jenkins
RUN npm install -g selenium-side-runner chromedriver edgedriver geckodriver iedriver --unsafe-perm=true --allow-root

# Install Jmeter
RUN  echo "http://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories \
    && curl -L https://archive.apache.org/dist/jmeter/binaries/apache-jmeter-5.3.tgz >  /tmp/jmeter.tgz \
    && tar -xvf /tmp/jmeter.tgz -C /opt \
    && rm /tmp/jmeter.tgz \
    && curl -L https://jmeter-plugins.org/get/ > /opt/apache-jmeter-5.3/lib/ext/jmeter-plugins-manager.jar \
    && curl -L https://repo1.maven.org/maven2/kg/apc/cmdrunner/2.2/cmdrunner-2.2.jar > /opt/apache-jmeter-5.3/lib/cmdrunner-2.2.jar

# Generate Plugins Manager command line utility:
RUN java -cp /opt/apache-jmeter-5.3/lib/ext/jmeter-plugins-manager.jar org.jmeterplugins.repository.PluginManagerCMDInstaller

# Install the WebDriver Sampler plugin:
RUN /opt/apache-jmeter-5.3/bin/./PluginsManagerCMD.sh install jpgc-webdriver

ENV PATH $PATH:/opt/apache-jmeter-5.3/bin

############################################
# Configure Jenkins
############################################
# Jenkins settings
COPY config/*.xml ${REF}/

# Jenkins Settings, i.e. Maven, Groovy, ...
COPY config/hudson.tasks.Maven.xml ${REF}/hudson.tasks.Maven.xml
COPY config/hudson.plugins.groovy.Groovy.xml ${REF}/hudson.plugins.groovy.Groovy.xml
COPY config/maven-global-settings-files.xml ${REF}/org.jenkinsci.plugins.configfiles.GlobalConfigFiles.xml

# SSH Keys & Credentials
COPY config/ssh-keys/cd-demo ${REF}/.ssh/id_rsa
COPY config/ssh-keys/cd-demo.pub ${REF}/.ssh/id_rsa.pub

# tell Jenkins that no banner prompt for pipeline plugins is needed
# see: https://github.com/jenkinsci/docker#preinstalling-plugins
# RUN echo 2.0 > ${REF}/jenkins.install.UpgradeWizard.state
