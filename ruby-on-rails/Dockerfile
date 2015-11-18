FROM debian:jessie

RUN apt-get update \
	&& apt-get install -y --no-install-recommends \
		bzip2 \
		ca-certificates \
		curl \
		libffi-dev \
		libgdbm3 \
		libssl-dev \
		libyaml-dev \
		procps \
		zlib1g-dev \
	&& rm -rf /var/lib/apt/lists/*

ENV RUBY_MAJOR 2.2
ENV RUBY_VERSION 2.2.3
ENV RUBY_DOWNLOAD_SHA256 df795f2f99860745a416092a4004b016ccf77e8b82dec956b120f18bdc71edce
ENV RUBYGEMS_VERSION 2.5.0

# skip installing gem documentation
RUN echo 'install: --no-document\nupdate: --no-document' >> "$HOME/.gemrc"

# some of ruby's build scripts are written in ruby
# we purge this later to make sure our final image uses what we just built
RUN buildDeps=' \
		autoconf \
		bison \
		gcc \
		libbz2-dev \
		libgdbm-dev \
		libglib2.0-dev \
		libncurses-dev \
		libreadline-dev \
		libxml2-dev \
		libxslt-dev \
		make \
		ruby \
	' \
	&& set -x \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends $buildDeps \
	&& mkdir -p /usr/src/ruby \
	&& curl -fSL -o ruby.tar.gz "http://cache.ruby-lang.org/pub/ruby/$RUBY_MAJOR/ruby-$RUBY_VERSION.tar.gz" \
	&& echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.gz" | sha256sum -c - \
	&& tar -xzf ruby.tar.gz -C /usr/src/ruby --strip-components=1 \
	&& rm ruby.tar.gz \
	&& cd /usr/src/ruby \
	&& autoconf \
	&& ./configure --disable-install-doc \
	&& make -j"$(nproc)" \
	&& make install \
	&& gem update --system $RUBYGEMS_VERSION \
	&& rm -r /usr/src/ruby
	

# install things globally, for great justice
ENV GEM_HOME /usr/local/bundle
ENV PATH $GEM_HOME/bin:$PATH
ENV BUNDLER_VERSION 1.10.6
RUN gem install bundler --version "$BUNDLER_VERSION" \
	&& bundle config --global path "$GEM_HOME" \
	&& bundle config --global bin "$GEM_HOME/bin" \
	&& bundle config build.nokogiri --use-system-libraries

# don't create ".bundle" in all our apps
ENV BUNDLE_APP_CONFIG $GEM_HOME
# see update.sh for why all "apt-get install"s have to stay as one long line
RUN apt-get update \
	&& apt-get install -y nodejs --no-install-recommends \
	&& apt-get install -y mysql-client --no-install-recommends	

# see http://guides.rubyonrails.org/command_line.html#rails-dbconsole

ENV RAILS_VERSION 4.2.3

RUN gem install rails --version "$RAILS_VERSION"

RUN apt-get purge -y --auto-remove $buildDeps \
	&& rm -rf /var/lib/apt/lists/*