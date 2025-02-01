FROM jenkins/jenkins:lts

USER root

RUN apt-get update && apt-get install -y lsb-release

RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg

RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list

RUN apt-get update && apt-get install -y docker-ce-cli

RUN apt update && apt install tzdata python3 python3-pip python3-pytest python3-flask python3-flake8 python3-coverage wget -y

RUN wget https://downloads.apache.org/jmeter/binaries/apache-jmeter-5.6.3.tgz  && \
    tar -xzf apache-jmeter-5.6.3.tgz -C /opt/ && \
    ln -s /opt/apache-jmeter-5.6.3/bin/jmeter /usr/local/bin/jmeter && \
    rm apache-jmeter-5.6.3.tgz

RUN pip install --no-cache-dir bandit --break-system-packages

ENV JMETER_HOME=/opt/apache-jmeter-5.6.3

ENV TZ="America/Mazatlan"

USER jenkins
