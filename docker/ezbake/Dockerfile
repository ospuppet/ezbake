FROM ruby:3.2-slim-bookworm

ARG version
ARG vcs_ref
ARG build_date
ARG source_url
ARG image_vendor
ARG image_user_name
ARG image_user_email

ENV LANG="en_US.utf8"
ENV EZBAKE_ALLOW_UNREPRODUCIBLE_BUILDS=true
ENV EZBAKE_NODEPLOY=true
ENV GEM_SOURCE=https://rubygems.org
ENV LEIN_PROFILES=ezbake
ENV USER_NAME="$image_user_name"
ENV USER_EMAIL="$image_user_email"

LABEL org.opencontainers.image.title="ezbake" \
      org.opencontainers.image.vendor="$image_vendor" \
      org.opencontainers.image.url="$source_url" \
      org.opencontainers.image.version="$version" \
      org.opencontainers.image.source="$source_url" \
      org.opencontainers.image.revision="$vcs_ref" \
      org.opencontainers.image.created="$build_date" \
      org.opencontainers.image.documentation="$source_url/docker/README.md" \
      org.opencontainers.image.description="Build custom packages for projects using ezbake, including PuppetServer and PuppetDB" \
      org.opencontainers.image.authors="$USER_NAME <$USER_EMAIL>" \
      org.opencontainers.image.licenses="Apache-2.0"

RUN rm -f /etc/apt/apt.conf.d/docker-clean; echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
  --mount=type=cache,target=/var/lib/apt,sharing=locked \
        apt-get -y update -q && \
        apt-get -y install --no-install-recommends build-essential rpm openjdk-17-jdk-headless git curl rsync

RUN git config --global user.name "$USER_NAME" && \
    git config --global user.email "$USER_EMAIL" && \
    git config --global advice.detachedHead false && \
    git config --global --add safe.directory /workspace && \
    git config --global --add safe.directory /ezbake

RUN curl --output /usr/local/bin/lein https://raw.githubusercontent.com/technomancy/leiningen/stable/bin/lein && \
    chmod 0755 /usr/local/bin/lein && \
    /usr/local/bin/lein

RUN gem install --no-doc bundler fpm && \
    mkdir /workspace /ezbake_src

COPY docker/ezbake/docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh

# Have the latest ezbake installed 
COPY . /ezbake_src
WORKDIR /ezbake_src
RUN lein clean && lein install

WORKDIR /
COPY docker/ezbake/Dockerfile /
RUN <<EOF
{
    echo "version=\"$version\"" ;
    echo "vcs_ref=\"$vcs_ref\"" ;
    echo "build_date=\"$build_date\"" ;
    echo "source_url=\"$source_url\"" ;
    echo "image_vendor=\"$image_vendor\"" ;
    echo "image_user_name=\"$image_user_name\"" ;
    echo "image_user_email=\"$image_user_email\"" ;
} > .docker_build_args
rm -rf /ezbake_src
EOF

ENTRYPOINT ["/docker-entrypoint.sh"]
