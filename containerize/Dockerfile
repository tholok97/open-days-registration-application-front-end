# Copyright (C) 2019, CERN
# This software is distributed under the terms of the GNU General Public
# Licence version 3 (GPL Version 3), copied verbatim in the file "LICENSE".
# In applying this license, CERN does not waive the privileges and immunities
# granted to it by virtue of its status as Intergovernmental Organization
# or submit itself to any jurisdiction.

# This docker file serves as an example. It was using a CentOS 7 image. In case you want to use this file for image build please point the following line to a CentOS 7 image. 

#################################
# action needed before use      # 
# please read the readme        #

FROM "$your_centos_image"
ARG REGISTRATION_API_URL="$your_backend_URL"
ARG RECAPTCHA_SITE_KEY="$your_google_recaptcha_site_key"
ARG RECAPTCHA_ENABLED="false"
ARG PIWIK_SITE_ID="$your_piwik_site_id"
ARG PIWIK_DOMAIN="$your_piwik_domain"
#################################

ARG DUMB_INIT_VERSION="1.2.2"
ARG DUMB_INIT_SHA256="37f2c1f0372a45554f1b89924fbb134fc24c3756efaedf11e07f599494e0eff9"
ENV REGISTRATION_API_URL=${REGISTRATION_API_URL}
ENV RECAPTCHA_SITE_KEY=${RECAPTCHA_SITE_KEY}
ENV RECAPTCHA_ENABLED=${RECAPTCHA_ENABLED}
ENV PIWIK_SITE_ID=${PIWIK_SITE_ID}
ENV PIWIK_DOMAIN=${PIWIK_DOMAIN}

# start bash session so startup script is going to be executed
SHELL ["/bin/bash", "-l", "-c"]

# install and update packages and 
# remove orphaned packages to lower the size of the image
# install maven

RUN yum install -y exclude httpd gettext sudo -q && \
  curl -sL https://rpm.nodesource.com/setup_10.x | sudo bash - && \
  yum install -y exclude nodejs -q && \
  yum clean all -q && rm -rf /var/cache/yum/* && \
  yum remove `package-cleanup --leaves` -q && \
  curl -s -L -o /usr/local/bin/dumb-init https://github.com/Yelp/dumb-init/releases/download/v${DUMB_INIT_VERSION}/dumb-init_${DUMB_INIT_VERSION}_amd64 && \
  echo "$DUMB_INIT_SHA256  /usr/local/bin/dumb-init" | sha256sum -c - && \
  chmod +x /usr/local/bin/dumb-init && \
  npm install -g @angular/cli

# copy resources into docker
COPY application /opt/application
COPY application-welcome /opt/application-welcome
COPY containerize/scripts /opt/scripts
COPY containerize/configuration/apache /opt/apache

#################################
# action needed before use      # 
# please read the readme        #

COPY configuration/web-root /opt/web
#################################

RUN mv /opt/apache/02-rewrite.conf /etc/httpd/conf.modules.d/02-rewrite.conf && \
  mv /opt/apache/opendays-app.conf /etc/httpd/conf.d/opendays-app.conf && \
  chmod a+x /opt/scripts/startup.sh && \
  rm -rf /var/www/html/* && \
  mv /opt/application/src/index.html /opt/index.html && \
  envsubst < /opt/index.html >> /opt/application/src/index.html && \
  mv /opt/application/src/app/SharedObjects/constants/generalSettings.ts /opt/generalSettings.ts && \
  envsubst < /opt/generalSettings.ts >> /opt/application/src/app/SharedObjects/constants/generalSettings.ts && \
  cd /opt/application-welcome && \
  npm install --save-dev @angular-devkit/build-angular && \
  npm run ng build -- --prod --output-path=/var/www/html/ && \ 
  cd /opt/application && \
  npm install --save-dev @angular-devkit/build-angular && \
  npm run build:en -- --output-path=/var/www/html/en/ --base-href /en/ && \
  npm run build:fr -- --output-path=/var/www/html/fr/ --base-href /fr/ && \
  mv /opt/apache/.htaccess /var/www/html/.htaccess && \
  mv /opt/apache/.htaccess_fr /var/www/html/fr/.htaccess && \
  mv /opt/apache/.htaccess_en /var/www/html/en/.htaccess && \
  mv /opt/web/* /var/www/html/ && \  
  rm -rf /opt/application/dist

EXPOSE 80

ENTRYPOINT ["/usr/local/bin/dumb-init","--"]

CMD ["/opt/scripts/startup.sh"] 
