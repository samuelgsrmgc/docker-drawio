FROM node:lts-slim

LABEL maintainer="JGraph Ltd"

# We don't download Puppeteer's bundled Chromium; point Puppeteer at the
# system google-chrome-stable installed below instead.
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true \
    PUPPETEER_EXECUTABLE_PATH=/usr/bin/google-chrome-stable

# From https://github.com/puppeteer/puppeteer/blob/main/docker/Dockerfile
# Install google-chrome-stable and fonts to support major charsets (Chinese, Japanese, Arabic, Hebrew, Thai and a few others).
# Note: PUPPETEER_EXECUTABLE_PATH above makes Puppeteer use this system Chrome
# rather than looking for a bundled copy it never downloaded.
RUN apt-get update \
    && apt-get install -y wget gnupg git curl \
    && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | gpg --dearmor -o /usr/share/keyrings/googlechrome-linux-keyring.gpg \
    && sh -c 'echo "deb [arch=amd64 signed-by=/usr/share/keyrings/googlechrome-linux-keyring.gpg] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' \
    && apt-get update \
    && apt-get install -y google-chrome-stable fonts-ipafont-gothic fonts-wqy-zenhei fonts-thai-tlwg fonts-khmeros fonts-kacst fonts-freefont-ttf libxss1 dbus dbus-x11 \
        --no-install-recommends \
    && service dbus start \
    && wget http://ftp.de.debian.org/debian/pool/contrib/m/msttcorefonts/ttf-mscorefonts-installer_3.8_all.deb \
    && apt install -y --no-install-recommends \
        ./ttf-mscorefonts-installer_3.8_all.deb \
    && apt install -y --no-install-recommends minizip \
    && groupadd -r pptruser && useradd -rm -g pptruser -G audio,video pptruser \
    && cd /home/pptruser \
    && git clone https://github.com/jgraph/draw-image-export2.git \
    # Root group (GID 0) gets owner-equivalent permissions so the server can
    # run under an arbitrary UID carrying GID 0 (OpenShift default; --user
    # <uid>:0 / group_add / supplementalGroups elsewhere). Chrome creates its
    # profile dir (userDataDir) inside the app directory at render time, so
    # read access alone is not enough. [jgraph/docker-drawio#186]
    && chown -R pptruser:0 /home/pptruser \
    && chmod -R g=u /home/pptruser \
    # Only purge git (a build-time clone tool). Do NOT purge wget: it is a
    # runtime dependency of google-chrome-stable, so removing it uninstalls
    # Chrome and leaves PUPPETEER_EXECUTABLE_PATH dangling -> HTTP 500 on render.
    && apt-get remove -y --purge git \
    # Fail the build early if the cleanup ever removes Chrome again.
    && test -x /usr/bin/google-chrome-stable \
    && rm -rf /var/lib/apt/lists/*

USER pptruser

ENV DBUS_SESSION_BUS_ADDRESS autolaunch:

WORKDIR /home/pptruser/draw-image-export2

RUN npm install

# Chrome resolves paths under $HOME at launch; an arbitrary runtime UID has no
# passwd entry, so HOME falls back to '/', which is not writable, and the
# browser launch aborts. /tmp is writable for every UID. Set after 'npm install'
# so the build-time npm cache is not baked into the image at /tmp.
# [jgraph/docker-drawio#186]
ENV HOME=/tmp

EXPOSE 8000

CMD ["npm", "start"]