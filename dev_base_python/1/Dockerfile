######################################################################
# Dockerfile to build Python WSGI Application Containers
# Based on CentOS
######################################################################

# Set the base image to Centos. NOTE: By specifying the minor version tag, it will not receive updates!
FROM centos:centos7.2.1511

# Install support for overlayfs (the default in docker 1.13+)
RUN yum -y -q install yum-plugin-ovl --setopt=tsflags=nodocs && \
  \
# Update system, despite a warning against this: https://docs.docker.com/articles/dockerfile_best-practices/#run
# The official CentOS Dockerfiles repo recommends it: https://github.com/CentOS/CentOS-Dockerfiles/blob/master/nginx/centos7/Dockerfile#L10
  yum -q -y clean expire-cache && \
  yum -q -y update --setopt=tsflags=nodocs && \
  yum -q -y clean all

# Install dev tools to allow python modules with native code (like markupsafe) to compile
# Install IUS repo then install python3, pip3 and python34u-devel (for psycopg2 etc)
  RUN yum -q -y groupinstall "Development Tools" --skip-broken --setopt=tsflags=nodocs && \
  yum -q -y install wget --setopt=tsflags=nodocs && \
  yum install -q -y https://centos7.iuscommunity.org/ius-release.rpm  && \
  yum install -q -y python34u python34u-pip python34u-devel --setopt=tsflags=nodocs && \
  yum -q -y clean all

# To allow httpretty to install via pip3 (see https://github.com/pypa/pip/issues/1291)
ENV LC_ALL=en_US.UTF-8 \
\
# Force stdin, stdout and stderr to be totally unbuffered
  PYTHONUNBUFFERED=yes \
\
# Set the log level to DEBUG for development
  LOG_LEVEL=DEBUG
