# Need to execute as root.
#   sudo podman build -t my-unbound .
#   sudo podman run -d -p 1053:53/tcp -p 1053:53/udp --rm --name test-unbound localhost/my-unbound

FROM registry.access.redhat.com/ubi8/ubi-init

RUN dnf -y update

RUN dnf -y install less procps-ng iproute bind-utils

# groupadd fails in rootless mode
# https://github.com/containers/fuse-overlayfs/issues/183
RUN groupadd -r unbound \
 && useradd -r -g unbound -d /etc/unbound -s /sbin/nologin -c "Unbound DNS resolver" unbound \
 && dnf -y install unbound openssl

RUN ln -s /usr/lib/systemd/system/unbound.service /etc/systemd/system/multi-user.target.wants/unbound.service \
 && rm /etc/unbound/conf.d/example.com.conf \
 && rm /etc/unbound/keys.d/example.com.key \
 && rm /etc/unbound/local.d/block-example.com.conf

COPY my-server.conf /etc/unbound/conf.d/

USER root:unbound

RUN /usr/sbin/unbound-control-setup -d /etc/unbound/ \
 && /usr/sbin/unbound-checkconf

EXPOSE 53/udp 53/tcp
