FROM golang as builder

ARG VERSION=0.8.4

WORKDIR /tmp
RUN set -ex; \
	# wget -O src.tar.gz https://github.com/ajvb/kala/archive/v${VERSION}.tar.gz; \
	# tar -xzf src.tar.gz; \
	# mv kala-${VERSION} kala; \
	# Because of https://github.com/ajvb/kala/pull/250, temporary use my personal repo
	git clone https://github.com/up9cloud/kala.git; \
	cd kala; \
	CGO_ENABLED=0 go build -o /tmp/main

FROM zenika/alpine-chrome
# All things back to root: https://github.com/Zenika/alpine-chrome/blob/master/Dockerfile
USER root
RUN set -e; \
	apk add --no-cache \
	tini \
	bash \
	curl \
	rsync \
	openssh-client \
	bind-tools \
	jq \
# In the mean time, install all famous script languages
	python3 \
	php \
	nodejs nodejs-npm \
	ruby \
	lua5.3

# Essentials for scripts
ENV NODE_PATH=/usr/lib/node_modules
RUN set -e; \
	if [ "/usr/lib/node_modules" != "$(npm root -g)" ]; then \
		echo >&2 "NODE_PATH invalid" && exit 2; \
	fi
# Default playwright browsers path: https://github.com/microsoft/playwright/blob/master/src/utils/registry.ts#L185
# playwright: https://github.com/Zenika/alpine-chrome/blob/master/with-playwright/Dockerfile
# puppeteer: https://github.com/Zenika/alpine-chrome/blob/master/with-puppeteer/Dockerfile
ENV PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1 \
	PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH=/usr/bin/chromium-browser \
	PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=1 \
	PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser
RUN set -e; \
	npm -g install \
		playwright-chromium \
		puppeteer \
		puppeteer-extra \
		puppeteer-extra-plugin-stealth \
		puppeteer-extra-plugin-user-preferences \
		puppeteer-extra-plugin-user-data-dir

ENTRYPOINT ["tini", "--"]
COPY --from=builder /tmp/main /usr/local/bin/kala
COPY --from=builder /tmp/kala/webui /app/webui
WORKDIR /app
# change default db root path to /tmp, let user mount the volume easier
CMD ["kala", "serve", "--bolt-path=/tmp"]
EXPOSE 8000
