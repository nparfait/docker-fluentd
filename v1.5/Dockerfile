FROM fluent/fluentd:v1.5.0-debian-1.0
MAINTAINER Baznai Cloud <info@banzaicloud.com>

RUN mkdir -p /fluentd/etc /fluentd/plugins
ONBUILD COPY plugins /fluentd/plugins/

USER root

RUN buildDeps="make gcc g++ libc-dev ruby-dev zlib1g-dev libz-dev git" \
  && apt-get update \
  && apt-get install -y --no-install-recommends $buildDeps \
  && gem install \
        specific_install \
        fluent-plugin-remote-syslog \
        fluent-plugin-kubernetes_metadata_filter \
        fluent-plugin-webhdfs \
        fluent-plugin-elasticsearch \
        fluent-plugin-prometheus \
        fluent-plugin-s3 \
        fluent-plugin-rewrite-tag-filter \
        fluent-plugin-azurestorage \
        fluent-plugin-oss 


RUN gem specific_install -l https://github.com/banzaicloud/fluent-plugin-gcs.git

RUN rm -rf /tmp/loki \
    && git clone --depth 1 https://github.com/grafana/loki.git /tmp/loki \
    && cd /tmp/loki/fluentd/fluent-plugin-loki \
    && sed -i "s|^  spec.files.*|  spec.files = Dir.glob('{bin,lib}/**/*')|" fluent-plugin-loki.gemspec \
    && gem build fluent-plugin-loki.gemspec \
    && gem install fluent-plugin-loki-*.gem \
    && cd /tmp \
    && rm -rf /tmp/loki
    
RUN gem sources --clear-all \
  && apt-get purge -y --auto-remove \
                  -o APT::AutoRemove::RecommendsImportant=false \
                  $buildDeps \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* \
         /home/fluent/.gem/ruby/2.3.0/cache/*.gem \
         /var/lib/gems/2.3.0/cache/*.gem \
  && apt-get purge -y $buildDeps

ADD fluent.conf /fluentd/etc/fluent.conf
WORKDIR  /home/fluent/

EXPOSE 24284