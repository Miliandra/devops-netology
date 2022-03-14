FROM centos:7
RUN cd /opt &&\
    groupadd elasticsearch && \
    useradd -c "elasticsearch" -g elasticsearch elasticsearch &&\
    mkdir /var/lib/data && chmod -R 777 /var/lib/data &&\
    yum -y install wget perl-Digest-SHA && \
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.1.0-linux-x86_64.tar.gz && \
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.1.0-linux-x86_64.tar.gz.sha512 && \
    shasum -a 512 -c elasticsearch-8.1.0-linux-x86_64.tar.gz.sha512 && \
    tar -xzf elasticsearch-8.1.0-linux-x86_64.tar.gz && \
    rm elasticsearch-8.1.0-linux-x86_64.tar.gz elasticsearch-8.1.0-linux-x86_64.tar.gz.sha512 && \
    chown -R elasticsearch:elasticsearch /opt/elasticsearch-8.1.0

USER elasticsearch
WORKDIR /opt/elasticsearch-8.1.0
ENV PATH=$PATH:/opt/elasticsearch-8.1.0/bin
COPY elasticsearch.yml config/
EXPOSE 9200
EXPOSE 9300
ENTRYPOINT ["elasticsearch"]
