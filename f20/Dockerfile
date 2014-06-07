# https://index.docker.io/_/fedora/
FROM fedora:20
RUN yum install -y bzr gcc git git-hg golang; yum clean all
ADD gotest /
RUN useradd user
