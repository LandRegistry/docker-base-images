######################################################################
# Dockerfile to build Java Application Containers
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

# Install Java and zip file support
# Also install make so the makefile can be used
RUN yum -q -y install java-1.8.0-openjdk-devel unzip wget which make --setopt=tsflags=nodocs && \
  yum -q -y clean all

# Explicitly expose the port set below
EXPOSE 8080

# Port for frameworks such as spark
# Default to 8080, docker-compose can change this to the reserved port for this app
ENV PORT=8080 \
  \
# Set the log level to DEBUG for development
  LOG_LEVEL=DEBUG \
  \
# Set commit (for the health route) to be local for development
  COMMIT=LOCAL \
  \
  # Gradle stuff
  GRADLE_HOME=/opt/gradle/latest \
  GRADLE_INSTALLED_VERSION='3.3'
# Got to be separate to pick up previously set var
ENV PATH $GRADLE_HOME/bin:$PATH

# Install gradle
RUN wget -q https://downloads.gradle.org/distributions/gradle-3.3-all.zip && \
  unzip -oqd /opt/gradle gradle-3.3-all.zip && \
  ln -sfn gradle-3.3 /opt/gradle/latest && \
  rm -f gradle-3.3-all.zip

# Default directory to run the app from
WORKDIR /src

# Run gradle using the compiled classes (not creating a fat jar for deployment)
CMD ["gradle", "run"]