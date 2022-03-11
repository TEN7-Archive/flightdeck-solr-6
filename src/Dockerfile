FROM alpine:3.15
MAINTAINER tess@ten7.com

# Update the package list and install Ansible.
RUN apk -U upgrade &&\
    apk add --update --no-cache ansible-core && \
    rm -rf /tmp/* \
           /var/cache/apk/*

# Copy the Ansible configuration files
COPY ansible-hosts /etc/ansible/hosts
COPY ansible.cfg /etc/ansible/ansible.cfg
COPY ansible /ansible

# Run the build.
RUN ansible-galaxy install -fr /ansible/requirements.yml && \
    ansible-playbook -i /ansible/inventories/all.ini /ansible/build.yml

# Configure the runtime environment of the container.
VOLUME /data
EXPOSE 8983

# Switch to the service account and set SOLR_PID_DIR
USER solr
ENV SOLR_PID_DIR /home/solr

# Set the entrypoint and default command.
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
CMD ["docker-cmd.sh", "/opt/solr/bin/solr-foreground.sh"]
