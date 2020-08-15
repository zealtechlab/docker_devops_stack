FROM alpine:latest

LABEL Prabhakaran Sampath <prabhakaran@zealtechlab.co.in>

ENV DISPLAY :99
ENV RESOLUTION 1366x768x24

RUN  echo "http://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories \
    && apk add --no-cache curl xfce4-terminal xvfb x11vnc xfce4 openjdk8-jre bash xrdp \
    && curl -L https://archive.apache.org/dist/jmeter/binaries/apache-jmeter-5.3.tgz >  /tmp/jmeter.tgz \
    && tar -xvf /tmp/jmeter.tgz -C /opt \
    && rm /tmp/jmeter.tgz \
    && curl -L https://jmeter-plugins.org/get/ > /opt/apache-jmeter-5.3/lib/ext/jmeter-plugins-manager.jar \
    && curl -L https://repo1.maven.org/maven2/kg/apc/cmdrunner/2.2/cmdrunner-2.2.jar > /opt/apache-jmeter-5.3/lib/ \
    && echo "[Globals]" > /etc/xrdp/xrdp.ini \
    && echo "bitmap_cache=true" >> /etc/xrdp/xrdp.ini \
    && echo "bitmap_compression=true" >> /etc/xrdp/xrdp.ini \
    && echo "autorun=jmeter" >> /etc/xrdp/xrdp.ini \
    && echo "[jmeter]" >> /etc/xrdp/xrdp.ini \
    && echo "name=jmeter" >> /etc/xrdp/xrdp.ini \
    && echo "lib=libvnc.so" >> /etc/xrdp/xrdp.ini \
    && echo "ip=localhost" >> /etc/xrdp/xrdp.ini \
    && echo "port=5900" >> /etc/xrdp/xrdp.ini \
    && echo "username=jmeter" >> /etc/xrdp/xrdp.ini \
    && echo "password=" >> /etc/xrdp/xrdp.ini

# Generate Plugins Manager command line utility:
RUN java -cp /opt/apache-jmeter-5.3/lib/ext/jmeter-plugins-manager.jar org.jmeterplugins.repository.PluginManagerCMDInstaller

# Install the WebDriver Sampler plugin:
RUN /opt/apache-jmeter-5.3/bin/./PluginsManagerCMD.sh install jpgc-webdriver

EXPOSE 5900
EXPOSE 3389

CMD ["bash", "-c", "rm -f /tmp/.X99-lock && rm -f /var/run/xrdp.pid\
 && nohup bash -c \"/usr/bin/Xvfb :99 -screen 0 ${RESOLUTION} -ac +extension GLX +render -noreset && export DISPLAY=99 > /dev/null 2>&1 &\"\
  && nohup bash -c \"startxfce4 > /dev/null 2>&1 &\"\
   && nohup bash -c \"x11vnc -xkb -noxrecord -noxfixes -noxdamage -display :99 -forever -bg -nopw -rfbport 5900 > /dev/null 2>&1\"\
    && nohup bash -c \"xrdp > /dev/null 2>&1\"\
     && nohup bash -c \"/opt/apache-jmeter-5.3/bin/./jmeter -Jjmeter.laf=CrossPlatform > /dev/null 2>&1 &\"\
       && tail -f /dev/null"]