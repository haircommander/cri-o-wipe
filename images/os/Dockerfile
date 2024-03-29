FROM fedora:29 AS build

COPY --from=registry.svc.ci.openshift.org/openshift/origin-v4.2:machine-os-content /srv/ /srv/
RUN set -x && yum install -y ostree yum-utils selinux-policy-targeted && \
    commit=$( find /srv -name *.commit | sed -Ee 's|.*objects/(.+)/(.+)\.commit|\1\2|' | head -1 ) && \
    mkdir /tmp/working && cd /tmp/working && \
    yumdownloader --enablerepo=built --disablerepo=fedora,fedora-modular,updates,updates-modular --destdir=/tmp/rpms cri-o && \
    for i in $(find /tmp/rpms/ -name cri-o* -iname *.rpm); do echo "Extracting $i ..."; rpm2cpio $i | cpio -div; done && \
    mv etc usr/ && \
    mkdir -p /tmp/tmprootfs/etc && \
    ostree --repo=/srv/repo checkout -U $commit --subpath /usr/etc/selinux /tmp/tmprootfs/etc/selinux && \
    ostree --repo=/srv/repo commit --parent=$commit --tree=ref=$commit --tree=dir=. \
        --selinux-policy /tmp/tmprootfs \
        -s "cri-o-ci-dev overlay RPMs" --branch=cri-o-ci-dev

FROM scratch
COPY --from=build /srv/ /srv/
