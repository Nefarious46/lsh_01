FROM rockylinux/rockylinux:9-ubi-micro AS runtime_container

FROM rockylinux/rockylinux:9-ubi AS assembly_container

RUN dnf install --assumeyes rpm-build java-25-openjdk-devel java-25-openjdk-jmods maven shadow-utils

COPY rpm/*.rpm /rpm/

RUN groupadd --gid 1001 srv-lsh_01 && \
    useradd --uid 1001 \
            --gid srv-lsh_01 \
            --home-dir /opt/teragrep/lsh_01 \
            --no-create-home \
            --shell /sbin/nologin \
            --comment "Service User" \
            srv-lsh_01

RUN  dnf install --assumeyes /rpm/*.rpm

COPY container/microjre.pom.xml /container/

WORKDIR /container

RUN mvn --batch-mode --file microjre.pom.xml clean package

RUN mkdir --parents /sysroot

COPY --from=runtime_container / /sysroot

RUN dnf upgrade --releasever 9 --setopt install_weak_deps=false --nodocs --installroot /sysroot --assumeyes

RUN  dnf install --releasever 9 --setopt install_weak_deps=false --nodocs --installroot /sysroot --assumeyes /container/target/rpm/com.teragrep-lsh_01_microjre/RPMS/x86_64/com.teragrep-lsh_01_microjre-*.rpm /rpm/com.teragrep-lsh_01-*.rpm

RUN dnf --installroot /sysroot clean all

RUN mkdir -p /scratch-etc && \
    echo "srv-lsh_01:x:1001:" > /scratch-etc/group && \
    echo "srv-lsh_01:x:1001:1001:Service User:/app:/sbin/nologin" > /scratch-etc/passwd

FROM scratch

COPY --from=assembly_container /sysroot /

WORKDIR  /opt/teragrep/lsh_01

COPY --from=assembly_container /scratch-etc/passwd /etc/passwd
COPY --from=assembly_container /scratch-etc/group /etc/group

COPY --from=assembly_container --chown=1001:1001 /opt/teragrep/lsh_01 /opt/teragrep/lsh_01

USER srv-lsh_01

COPY docker/docker-entrypoint.sh /docker-entrypoint.sh

ENTRYPOINT ["/bin/sh", "-c", "exec /docker-entrypoint.sh"]
