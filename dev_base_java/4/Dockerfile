######################################################################
# Dockerfile to build Java Application Containers
# Based on CentOS
######################################################################

# Set the base image to Centos. NOTE: By specifying the minor version tag, it will not receive updates!
FROM centos:centos7.4.1708

# Install support for overlayfs (the default in docker 1.13+)
# Update system, despite a warning against this: https://docs.docker.com/articles/dockerfile_best-practices/#run
# The official CentOS Dockerfiles repo recommends it: https://github.com/CentOS/CentOS-Dockerfiles/blob/master/nginx/centos7/Dockerfile#L10
RUN yum -y -q install yum-plugin-ovl --setopt=tsflags=nodocs && \
  yum -q -y clean expire-cache && \
  yum -q -y update --setopt=tsflags=nodocs && \
  yum -q -y clean all

# Install Java and zip file support
# Also install make so the makefile can be used
RUN yum -q -y install java-1.8.0-openjdk-devel-1.8.0.144-0.b01.el7_4 unzip wget which make --setopt=tsflags=nodocs && \
  yum -q -y clean all

# Explicitly expose the port set below
EXPOSE 8080

# ******* THE FOLLOWING ENV VARS WILL BE SET BY WEBOPS IN REAL ENVIRONMENTS *******

# The port for the server to listen on
ENV PORT 8080

# Will be used in real environments
ENV COMMIT LOCAL

# ******* END *******

ENV GRADLE_HOME=/opt/gradle/latest \
  GRADLE_INSTALLED_VERSION='4.2.1'
# Got to be separate to pick up previously set var
ENV PATH $GRADLE_HOME/bin:$PATH

# Install gradle
RUN wget -q https://downloads.gradle.org/distributions/gradle-4.2.1-all.zip && \
  unzip -oqd /opt/gradle gradle-4.2.1-all.zip && \
  ln -sfn gradle-4.2.1 /opt/gradle/latest && \
  rm -f gradle-4.2.1-all.zip

# Default directory to run the app from
WORKDIR /src

# We're changing the command to create a jar then run it to prevent gradle running in the background using hundreds of megs of ram.
# Any apps that use this as a base must import the shadowjar plugin into the build.gradle
CMD gradle shadowJar --no-daemon && java -jar $(find /src/build/libs -name '*.jar' | head -1)

# Set locale to be en_GB, this prevents UnicodeDecodeError and other random issues that might occur due to the lack of one
RUN localedef -c -i en_GB -f UTF-8 en_GB.UTF-8
ENV LANG='en_GB.UTF-8' \
    LANGUAGE='en_GB:en' \
    LC_ALL='en_GB.UTF-8'