FROM rabbitmq:3.8-alpine

RUN set -ex \
    && sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# Clean up
RUN rm -rf /var/cache/apk/* && \
    rm -rf /tmp/*

RUN rabbitmq-plugins enable --offline rabbitmq_management

EXPOSE 4369 5671 5672 15671 15672 25672

#   rabbitmqctl add_vhost industrious \
#   && rabbitmqctl set_permissions -p industrious tim ".*" ".*" ".*" \
#   && rabbitmqctl add_vhost cqgg-jobs \
#   && rabbitmqctl set_permissions -p cqgg-jobs tim ".*" ".*" ".*" \
#   && sed -i 's/true/false/' /etc/rabbitmq/conf.d/management_agent.disable_metrics_collector.conf \
#   && rabbitmqctl stop