FROM alpine:3.16 as builder

ENV LOOP_VERSION=v0.19.1-beta

WORKDIR /opt

RUN apk --no-cache add \
    bash \
    ca-certificates \
    gnupg \
    perl-utils

RUN wget https://github.com/lightninglabs/loop/releases/download/${LOOP_VERSION}/loop-linux-amd64-${LOOP_VERSION}.tar.gz
RUN wget https://github.com/lightninglabs/loop/releases/download/${LOOP_VERSION}/manifest-${LOOP_VERSION}.txt
RUN wget https://github.com/lightninglabs/loop/releases/download/${LOOP_VERSION}/manifest-${LOOP_VERSION}.txt.sig

RUN gpg  --keyserver hkps://keyserver.ubuntu.com --recv-keys DE23E73BFA8A0AD5587D2FCDE80D2F3F311FD87E && \
    gpg --verify manifest-${LOOP_VERSION}.txt.sig manifest-${LOOP_VERSION}.txt && \
    cat  manifest-${LOOP_VERSION}.txt | grep -q "$(shasum -a 256 loop-linux-amd64-${LOOP_VERSION}.tar.gz)"

RUN tar -xvzf loop-linux-amd64-${LOOP_VERSION}.tar.gz

FROM alpine:3.16 as final

ENV LOOP_VERSION=v0.19.1-beta
ENV PUID=10003
ENV PGID=10003

COPY --from=builder /opt/loop-linux-amd64-${LOOP_VERSION}/loop /usr/local/bin/
COPY --from=builder /opt/loop-linux-amd64-${LOOP_VERSION}/loopd /usr/local/bin/

RUN apk --no-cache add \
    su-exec \
    shadow

RUN addgroup --system --gid ${PGID} loop
RUN adduser loop --system --uid ${PUID} --home /loop -G loop

COPY ./entrypoint.sh /entrypoint.sh

EXPOSE 12010 8281

ENTRYPOINT [ "/entrypoint.sh" ]

CMD ["loopd"]
