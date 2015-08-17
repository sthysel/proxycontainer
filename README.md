# proxycontainer

squid and devpi docker based proxies for use with images that can make use of the using on-build triggers

```
# Determine build host
ONBUILD RUN netstat -nr | grep '^0\.0\.0\.0' | awk '{print $2}' > ${BUILD_HOST_FILE}

# squid proxy if available
ONBUILD RUN echo "HEAD /" | nc -q -1 -n -v  `cat ${BUILD_HOST_FILE}` 3128 | grep squid-deb-proxy \
  && (echo "Acquire::http::Proxy \"http://$(cat ${BUILD_HOST_FILE}):3128\";" > ${APT_PROXY_CONF}) \
  && (echo "Acquire::http::Proxy::ppa.launchpad.net DIRECT;" >> ${APT_PROXY_CONF}) \
  || echo "No squid-deb-proxy detected on docker host"

# devpi if available
ONBUILD RUN echo -en "HEAD /\n\n\n" | nc -q 1 -v `cat ${BUILD_HOST_FILE}` 3141 | grep Devpi \
  && mkdir -p ${PIP_CONF_DIR} \
  && (echo "[global]" > ${PIP_CONF_DIR}/pip.conf) \
  && (echo "timeout = 60" >> ${PIP_CONF_DIR}/pip.conf) \
  && (echo "index-url = http://$(cat ${BUILD_HOST_FILE}):3141/root/pypi/" >> ${PIP_CONF_DIR}/pip.conf) \
  && (echo "trusted-host = $(cat ${BUILD_HOST_FILE})" >> ${PIP_CONF_DIR}/pip.conf) \
  && (echo "no-cache-dir = true" >> ${PIP_CONF_DIR}/pip.conf) \
  && (echo "cache-dir = none" >> ${PIP_CONF_DIR}/pip.conf) \
  || echo "No devpi detected on docker host"
  ```
