FROM ubuntu:16.04

USER root

RUN apt-get update -y
RUN apt-get upgrade -y

RUN apt-get install -y software-properties-common python-software-properties apt-utils
RUN add-apt-repository -y ppa:git-core/ppa
RUN add-apt-repository -y ppa:ubuntu-toolchain-r/test
RUN apt-get install -y python2.7 python2.7-dev python-pip python-apt python-yaml \
    python-jinja2 build-essential sudo git-core libmysqlclient-dev libffi-dev libssl-dev \
    curl libxml2-dev libxslt1-dev python-dev libxmlsec1-dev libfreetype6-dev swig gcc g++

RUN pip install --upgrade pip==8.1.2

ENV PATH="/usr/local/bin:${PATH}"

RUN pip install setuptools==24.0.3
RUN pip install virtualenv==15.0.2

RUN /usr/local/bin/virtualenv /tmp/bootstrap

ENV PATH="/tmp/bootstrap/bin:${PATH}"

RUN git clone https://github.com/raccoongang/configuration.git /tmp/configuration && cd /tmp/configuration \
    && git checkout ficus-rg && git pull && make requirements
RUN pip install -r /tmp/configuration/requirements.txt

RUN cd /tmp/configuration/playbooks/edx-east
RUN /tmp/bootstrap/bin/ansible-playbook edx_ansible.yml -i '127.0.0.1,' -c local -e "configuration_version=ficus-rg" -e "edx_ansible_source_repo=https://github.com/raccoongang/configuration.git"
RUN /tmp/bootstrap/bin/ansible-playbook jenkins_worker.yml -i '127.0.0.1,' -c local

RUN rm -rf /tmp/ansible
RUN rm -rf /tmp/configuration
RUN rm -rf /tmp/bootstrap
RUN rm -rf "${HOME}/.ansible"
