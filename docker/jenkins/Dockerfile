FROM ubuntu:16.04

USER root

RUN apt-get update -y
RUN apt-get upgrade -y

RUN apt-get install -y software-properties-common python-software-properties apt-utils
RUN add-apt-repository -y ppa:git-core/ppa
RUN add-apt-repository -y ppa:ubuntu-toolchain-r/test
RUN apt-get install -y python2.7 python2.7-dev python-pip python-apt python-yaml openssh-server\
    python-jinja2 build-essential sudo git-core libmysqlclient-dev libffi-dev libssl-dev \
    curl libxml2-dev libxslt1-dev python-dev libxmlsec1-dev libfreetype6-dev swig gcc g++

RUN pip install --upgrade pip==8.1.2

ENV PATH="/usr/local/bin:${PATH}"

RUN pip install setuptools==24.0.3
RUN pip install virtualenv==15.0.2

RUN /usr/local/bin/virtualenv /tmp/bootstrap

ENV PATH="/tmp/bootstrap/bin:${PATH}"

RUN git clone https://github.com/python-squad/configuration.git /tmp/configuration && cd /tmp/configuration \
    && git checkout test-ficus-rg && git pull && make requirements

RUN /tmp/bootstrap/bin/ansible-playbook /tmp/configuration/playbooks/edx-east/edx_ansible.yml -i 'localhost,' \
    -c local -e "configuration_version=ficus-rg" -e "edx_ansible_source_repo=https://github.com/python-squad/configuration.git"
RUN /tmp/bootstrap/bin/ansible-playbook /tmp/configuration/playbooks/edx-east/jenkins_worker.yml -i 'localhost,' \
    -c local

RUN rm -rf /tmp/ansible
RUN rm -rf /tmp/configuration
RUN rm -rf /tmp/bootstrap
RUN rm -rf "${HOME}/.ansible"
