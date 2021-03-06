######################################################################
# Dockerfile to build Ruby Application Containers
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

# Install rbenv/phantomjs system dependencies
RUN yum install -y -q gcc-c++ patch readline readline-devel zlib zlib-devel libyaml-devel libffi-devel \
  openssl-devel make bzip2 autoconf automake libtool bison iconv-devel git-core wget fontconfig
# Everything between here and the phantomjs install is shamelessly ripped from
# https://github.com/docker-library/ruby/blob/master/2.3/Dockerfile
# with the apt-get bits removed
# skip installing gem documentation
RUN mkdir -p /usr/local/etc \
	&& { \
		echo 'install: --no-document'; \
		echo 'update: --no-document'; \
	} >> /usr/local/etc/gemrc
ENV RUBY_MAJOR 2.3
ENV RUBY_VERSION 2.3.3
ENV RUBY_DOWNLOAD_SHA256 241408c8c555b258846368830a06146e4849a1d58dcaf6b14a3b6a73058115b7
ENV RUBYGEMS_VERSION 2.6.8
# some of ruby's build scripts are written in ruby
#   we purge system ruby later to make sure our final image uses what we just built
RUN set -ex \
	\
	&& buildDeps=' \
		bison \
		libgdbm-dev \
		ruby \
	' \
	&& rm -rf /var/lib/apt/lists/* \
	\
	&& wget -O ruby.tar.gz "https://cache.ruby-lang.org/pub/ruby/$RUBY_MAJOR/ruby-$RUBY_VERSION.tar.gz" \
	&& echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.gz" | sha256sum -c - \
	\
	&& mkdir -p /usr/src/ruby \
	&& tar -xzf ruby.tar.gz -C /usr/src/ruby --strip-components=1 \
	&& rm ruby.tar.gz \
	\
	&& cd /usr/src/ruby \
	\
# hack in "ENABLE_PATH_CHECK" disabling to suppress:
#   warning: Insecure world writable dir
	&& { \
		echo '#define ENABLE_PATH_CHECK 0'; \
		echo; \
		cat file.c; \
	} > file.c.new \
	&& mv file.c.new file.c \
	\
	&& autoconf \
	&& ./configure --disable-install-doc \
	&& make -j"$(nproc)" \
	&& make install \
	\
	&& cd / \
	&& rm -r /usr/src/ruby \
	\
	&& gem update --system "$RUBYGEMS_VERSION"
ENV BUNDLER_VERSION 1.13.6
RUN gem install bundler --version "$BUNDLER_VERSION"
# install things globally, for great justice
# and don't create ".bundle" in all our apps
ENV GEM_HOME /usr/local/bundle
ENV BUNDLE_PATH="$GEM_HOME" \
	BUNDLE_BIN="$GEM_HOME/bin" \
	BUNDLE_SILENCE_ROOT_WARNING=1 \
	BUNDLE_APP_CONFIG="$GEM_HOME"
ENV PATH $BUNDLE_BIN:$PATH
RUN mkdir -p "$GEM_HOME" "$BUNDLE_BIN" \
	&& chmod 777 "$GEM_HOME" "$BUNDLE_BIN"
CMD [ "irb" ]
## Install Phantomjs
RUN export PHANTOM_JS="phantomjs-2.1.1-linux-x86_64" && \
  wget -q https://bitbucket.org/ariya/phantomjs/downloads/$PHANTOM_JS.tar.bz2  && \
  mv $PHANTOM_JS.tar.bz2 /usr/local/share/  && \
  cd /usr/local/share/  && \
  tar xvjf $PHANTOM_JS.tar.bz2  && \
  ln -sf /usr/local/share/$PHANTOM_JS/bin/phantomjs /usr/local/share/phantomjs  && \
  ln -sf /usr/local/share/$PHANTOM_JS/bin/phantomjs /usr/local/bin/phantomjs  && \
  ln -sf /usr/local/share/$PHANTOM_JS/bin/phantomjs /usr/bin/phantomjs

# Without this, phantomjs screenshots render without any text on the screen if your site is using or falling back to system fonts
RUN yum install -y -q fontconfig xorg-x11-fonts-Type1