# This image includes both the web app and the data import related scripts in
# one image. Because of historic reasons some of these scripts run java
# artifacts under the hood. It therefore currently makes more sense to have a
# single image supporting both instead of moving the scripts out to a separate
# container.
FROM maven:3.5.4 as build
COPY $PWD /cbioportal
WORKDIR /cbioportal
RUN mvn -DskipTests clean install

FROM openjdk:8-jre-slim
 
RUN mkdir -p /cbioportal

# install MYSQL build and runtime dependencies
# ignore update failures
RUN   apt-get update  \
      && apt-get install -y --no-install-recommends \
        gcc \
        default-libmysqlclient-dev \
        python3 \
        python3-pip \
        python3-setuptools \
        python3-dev \
        python3-wheel \
        && apt-get autoremove \
	&& rm -rf /var/lib/apt/lists/* 


COPY --from=build /cbioportal/requirements.txt /cbioportal/requirements.txt
RUN  pip3 install --no-cache-dir -r /cbioportal/requirements.txt


# copy over core jar and scripts
COPY --from=build /cbioportal/core /cbioportal/core
COPY --from=build /cbioportal/scripts /cbioportal/scripts

# add importer scripts to PATH for easy running in containers
RUN find /cbioportal/core/src/main/scripts/ -type f -executable \! -name '*.pl'  -print0 | xargs -0 -- ln -st /usr/local/bin

# put config files in this folder if you want to override config
ENV PORTAL_HOME=/cbioportal

# TODO: fix the workdir-dependent references to '../scripts/env.pl' and do this:
# RUN find $PWD/core/src/main/scripts/ -type f -executable \! \( -name env.pl -o -name envSimple.pl \)  -print0 | xargs -0 -- ln -st /usr/local/bin

