ARG DOCKER_REPOSITORY

FROM ${DOCKER_REPOSITORY}/tengine

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80 443 8443

STOPSIGNAL SIGTERM

CMD ["nginx", "-g", "daemon off;"]
