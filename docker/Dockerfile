FROM ubuntu:latest@sha256:0350de24c67dd14df508afb8bda9e2f7fa44647e6efffc9156f684a9255af478


# Contributors
LABEL author="Vincent Voyer <vincent@zeroload.net>"
LABEL maintainers="Serban Ghita <serbanghita@gmail.com>, David Catalan <catalan.david@gmail.com>"


# Env variables
ENV LC_ALL=C
ENV DISPLAY=:99
ENV SELENIUM_CONSOLE_URL=http://localhost:4444/wd/hub
ENV SCREEN_GEOMETRY=1024x768x16


# Expose Selenium web console port
EXPOSE 4444


# Update packages
RUN apt-get -qqy update


# Install commons
RUN apt-get -qqy install \
    apt-transport-https \
    ca-certificates \
    curl \
    jq \
    software-properties-common \
    sudo \
    openjdk-8-jre-headless \
    wget \
    xvfb \
    xfonts-100dpi \
    xfonts-75dpi \
    xfonts-scalable \
    xfonts-cyrillic


# Install Browser from vendor source
# Strategies copied from official Selenium Docker images (https://github.com/SeleniumHQ/docker-selenium)

## Install latest Firefox
ARG FIREFOX_VERSION=latest
RUN FIREFOX_DOWNLOAD_URL=$(if [ $FIREFOX_VERSION = "latest" ] || [ $FIREFOX_VERSION = "nightly-latest" ] || [ $FIREFOX_VERSION = "devedition-latest" ]; then echo "https://download.mozilla.org/?product=firefox-$FIREFOX_VERSION-ssl&os=linux64&lang=en-US"; else echo "https://download-installer.cdn.mozilla.net/pub/firefox/releases/$FIREFOX_VERSION/linux-x86_64/en-US/firefox-$FIREFOX_VERSION.tar.bz2"; fi) \
  && apt-get -qqy --no-install-recommends install firefox \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/* \
  && wget --no-verbose -O /tmp/firefox.tar.bz2 $FIREFOX_DOWNLOAD_URL \
  && apt-get -y purge firefox \
  && rm -rf /opt/firefox \
  && tar -C /opt -xjf /tmp/firefox.tar.bz2 \
  && rm /tmp/firefox.tar.bz2 \
  && mv /opt/firefox /opt/firefox-$FIREFOX_VERSION \
  && ln -fs /opt/firefox-$FIREFOX_VERSION/firefox /usr/bin/firefox

## Install latest Google Chrome
ARG CHROME_VERSION="google-chrome-stable"
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
  && echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list \
  && apt-get update -qqy \
  && apt-get -qqy install \
    ${CHROME_VERSION:-google-chrome-stable} \
  && rm /etc/apt/sources.list.d/google-chrome.list \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/*

## Chrome Launch Script Wrapper (forces no-sandbox so no need to use --privileged parameter)
COPY ./scripts/wrap_chrome_binary /opt/bin/wrap_chrome_binary
RUN /opt/bin/wrap_chrome_binary


# Install Node.js LTS
RUN curl -sL https://deb.nodesource.com/setup_8.x | bash -
RUN apt-get -qqy --no-install-recommends install \
  nodejs


# Add non-root user
RUN useradd seluser \
         --shell /bin/bash  \
         --create-home \
  && usermod -a -G sudo seluser \
  && echo 'ALL ALL = (ALL) NOPASSWD: ALL' >> /etc/sudoers \
  && echo 'seluser:secret' | chpasswd


# Install selenium-standalone
ARG DEBUG
RUN npm install -g selenium-standalone
RUN selenium-standalone install


ADD ./scripts/ /home/seluser/scripts


# Force using non root user to execute the image
USER seluser


# Display Selenium environment
RUN . /home/seluser/scripts/utils.sh && print_selenium_env


# Selenium server healthcheck
HEALTHCHECK --start-period=5s --interval=3s --retries=8 \
    CMD curl -qsf "$SELENIUM_CONSOLE_URL/status" | jq -r '.value.ready' | grep "true" || exit 1


ENTRYPOINT ["sh", "/home/seluser/scripts/start.sh"]
