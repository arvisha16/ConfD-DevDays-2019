FROM python:3

ARG CONFD_VERSION
ARG NSO_VERSION
ARG APP_NAME

ENV CONFD_VERSION=${CONFD_VERSION}
ENV NSO_VERSION=${NSO_VERSION}
ENV APP_NAME=${APP_NAME}
ENV DEBIAN_FRONTEND=noninteractive
ENV CONFD_DIR=/confd NCS_DIR=/nso
ENV LD_LIBRARY_PATH=/nso/lib:/confd/lib PYTHONPATH=/nso/src/ncs/pyapi
ENV PATH=/confd/bin:/nso/bin:$PATH

COPY nso-${NSO_VERSION}.linux.x86_64.installer.bin /tmp
WORKDIR /nso
RUN pip install --upgrade pip \
    && pip install --no-cache-dir paramiko lxml pexpect pytest \
    && apt-get update \
    && apt-get install -y --no-install-recommends libxml2-utils xsltproc emacs-nox default-jre htop sysstat \
    && /tmp/nso-${NSO_VERSION}.linux.x86_64.installer.bin /nso \
    && rm -rf examples.ncs doc

COPY confd-${CONFD_VERSION}.linux.x86_64.installer.bin /tmp
WORKDIR ${CONFD_DIR}
RUN ln -s libcrypto.so /usr/lib/x86_64-linux-gnu/libcrypto.so.1.0.0 \
    && /tmp/confd-${CONFD_VERSION}.linux.x86_64.installer.bin ${CONFD_DIR} \
    && rm -rf examples.confd doc

# Add the ConfD cryptography integration and C-library API source
ADD confd-${CONFD_VERSION}.libconfd.tar.gz /tmp

# Rebuild the CondfD crypto integration for libcrypto1.1
WORKDIR /tmp/confd-${CONFD_VERSION}/libconfd
RUN make install_crypto \
    && rm -rf /tmp/* /var/tmp/* \
    && apt-get autoremove -y \
    && apt-get clean

ADD ${APP_NAME}-confd.tar.gz /${APP_NAME}_confd
ADD ${APP_NAME}-nso.tar.gz /${APP_NAME}_nso

WORKDIR /${APP_NAME}_nso

EXPOSE 22 12022 12023 12032 12033 12042 12043 2022 4565 4575 4585 4569 8080 \
8008 8018 8028 8888 8898 8908
CMD [ "./run.sh" ]
