FROM postgres:9.6

RUN apt-get update && \
         apt-get -y install build-essential libreadline-dev zlib1g-dev flex bison libkeyutils-dev libnss3 default-jre vim

ADD 9.6.tar.gz /usr/lib/postgresql

ENV LD_LIBRARY_PATH=/usr/lib/postgresql/9.6/lib:$LD_LIBRARY_PATH

COPY docker-ag-entrypoint.sh /usr/local/bin/
RUN ln -s /usr/local/bin/docker-ag-entrypoint.sh / # backwards compat
RUN chmod +x /usr/local/bin/docker-ag-entrypoint.sh
ENTRYPOINT ["docker-ag-entrypoint.sh"]

EXPOSE 5432
CMD ["postgres"]

RUN useradd -ms /bin/bash stolon

ADD bin/stolon-keeper bin/stolon-sentinel bin/stolon-proxy bin/stolonctl /usr/local/bin/

RUN chmod +x /usr/local/bin/stolon-keeper /usr/local/bin/stolon-sentinel /usr/local/bin/stolon-proxy /usr/local/bin/stolonctl



