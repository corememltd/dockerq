FROM debian:9

ARG VCS_REF=dev
ARG BUILD_DATE=dev
ARG L64_SHA256=1439065d5baa0f7d152236733beea20745feb21a43cda62c33591db4fe7a40c5

LABEL	org.label-schema.schema-version="1.0" \
	org.label-schema.name=q \
	org.label-schema.description="Standalone q Docker Container" \
	org.label-schema.vendor="Kx" \
	org.label-schema.license="Apache-2.0" \
	org.label-schema.url="https://code.kx.com" \
	org.label-schema.version="${VERSION:-dev}" \
	org.label-schema.vcs-url="https://github.com/KxSystems/q.git" \
	org.label-schema.vcs-ref="$VCS_REF" \
	org.label-schema.build-date="$BUILD_DATE" \
	org.label-schema.docker.cmd="docker run kxsys/q"

SHELL ["/bin/sh", "-x", "-c"]

RUN passwd -d root

RUN apt-get update \
	&& apt-get -yy --option=Dpkg::options::=--force-unsafe-io upgrade \
	&& apt-get -yy --option=Dpkg::options::=--force-unsafe-io install --no-install-recommends \
		ca-certificates \
		curl \
		python-minimal \
		python-urllib3 \
		rlwrap \
		runit \
		unzip \
	&& apt-get clean \
	&& find /var/lib/apt/lists -type f -delete

ENV QHOME=/opt/kx/q

RUN mkdir -p $QHOME
RUN curl -f -o /tmp/l64.zip -L http://dls.kx.com/253h1j3s2941/download/l64.zip \
	&& [ "$L64_SHA256" = "$(sha256sum /tmp/l64.zip | cut -b1-64)" ] \
	&& unzip -d $QHOME /tmp/l64.zip \
	&& rm /tmp/l64.zip
COPY docker/q.wrapper /usr/local/bin/q
COPY docker/kc.lic.py /opt/kx/

COPY docker/init /init

ENTRYPOINT ["/init"]
CMD ["q"]
