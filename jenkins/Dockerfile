FROM jenkins/jenkins:lts

USER root
RUN apt-get update \
      && apt-get install -y sudo curl wget zip unzip\
      && apt-get install -y libltdl7\
      && apt-get install -y openjdk-8-jdk maven\
      && apt-get install -y git build-essential software-properties-common gnupg-agent\
      && rm -rf /var/lib/apt/lists/*

RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers

RUN sudo curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py\
      && sudo python3 get-pip.py && sudo pip install ansible

# getting the docker-cli
# --- Attention: docker.sock needs to be mounted as volume in docker-compose.yml
# see: https://issues.jenkins-ci.org/browse/JENKINS-35025
# see: https://get.docker.com/builds/
# see: https://wiki.jenkins-ci.org/display/JENKINS/CloudBees+Docker+Custom+Build+Environment+Plugin#CloudBeesDockerCustomBuildEnvironmentPlugin-DockerinDocker
RUN curl -sSL -o /bin/docker https://get.docker.io/builds/Linux/x86_64/docker-latest
RUN chmod +x /bin/docker
# RUN curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add - \
#       && sudo apt-key fingerprint 0EBFCD88\
#       && sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian \
#             $(lsb_release -cs) stable"\
#       && sudo apt-get update && sudo apt-get install -y docker-ce-cli\
#       && rm -rf /var/lib/apt/lists/*

USER jenkins

# installing specific list of plugins. see: https://github.com/jenkinsci/docker#preinstalling-plugins
COPY plugins.txt /usr/share/jenkins/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/plugins.txt

# Adding default Jenkins Seed Job
COPY jobs /usr/share/jenkins/ref/jobs/

############################################
# Configure Jenkins
############################################
# Jenkins settings
COPY config/*.xml /usr/share/jenkins/ref/

# Jenkins Settings, i.e. Maven, Groovy, ...
COPY config/hudson.tasks.Maven.xml /usr/share/jenkins/ref/hudson.tasks.Maven.xml
COPY config/hudson.plugins.groovy.Groovy.xml /usr/share/jenkins/ref/hudson.plugins.groovy.Groovy.xml
COPY config/maven-global-settings-files.xml /usr/share/jenkins/ref/org.jenkinsci.plugins.configfiles.GlobalConfigFiles.xml

# SSH Keys & Credentials
COPY config/ssh-keys/cd-demo /usr/share/jenkins/ref/.ssh/id_rsa
COPY config/ssh-keys/cd-demo.pub /usr/share/jenkins/ref/.ssh/id_rsa.pub

# tell Jenkins that no banner prompt for pipeline plugins is needed
# see: https://github.com/jenkinsci/docker#preinstalling-plugins
RUN echo 2.0 > /usr/share/jenkins/ref/jenkins.install.UpgradeWizard.state
