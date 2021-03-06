FROM ubuntu:14.04

ENV DEBIAN_FRONTEND noninteractive

ENV GALAXY_USER=galaxy \
GALAXY_UID=1450 \
GALAXY_GID=1450

MAINTAINER Björn A. Grüning, bjoern.gruening@gmail.com

RUN echo "force-unsafe-io" > /etc/dpkg/dpkg.cfg.d/02apt-speedup && \
    groupadd -r $GALAXY_USER -g $GALAXY_GID && \
    useradd -u $GALAXY_UID -m -r -g $GALAXY_USER -c "Galaxy user" $GALAXY_USER && \
    apt-get update -qq && apt-get install -y --no-install-recommends apt-transport-https \
        slurm-llnl slurm-llnl-torque slurm-drmaa-dev \
        python-pip python-psutil supervisor wget samtools && \
    /usr/sbin/create-munge-key && \
    touch /var/log/slurm-llnl/slurmctld.log /var/log/slurm-llnl/slurmd.log && \
    mkdir /tmp/slurm && pip install --upgrade supervisor virtualenv && \
    apt-get remove -y supervisor && mkdir /var/log/supervisor && \
    apt-get autoremove -y && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    mkdir -p /tmp/download && \
    wget -qO - https://get.docker.com/builds/Linux/x86_64/docker-17.05.0-ce.tgz | tar -xz -C /tmp/download && \
    mv /tmp/download/docker/docker /usr/bin/ && \
    rm -rf /tmp/download && \
    rm -rf ~/.cache/

ADD configure_slurm.py /usr/local/bin/configure_slurm.py
ADD munge.conf /etc/default/munge
ADD startup.sh /usr/bin/startup.sh
ADD supervisor_slurm.conf /etc/supervisor/conf.d/slurm.conf
RUN service munge start && service munge stop && \
    chmod +x /usr/bin/startup.sh && \
    locale-gen en_US.UTF-8 && dpkg-reconfigure locales

ENV GALAXY_DIR=/export/galaxy-central \
    SYMLINK_TARGET=/galaxy-central \
    SLURM_USER_NAME=galaxy \
    SLURM_UID=1450 \
    SLURM_GID=1450 \
    SLURM_PARTITION_NAME=work \
    SLURM_CLUSTER_NAME=Cluster \
    SLURMD_AUTOSTART=True \
    SLURMCTLD_AUTOSTART=True \
    SLURM_CONF_PATH=/export/slurm.conf \
    MUNGE_KEY_PATH=/export/munge.key \
    GALAXY_VENV=/galaxy_venv

ADD requirements.txt "$GALAXY_VENV"/
RUN chown -R $SLURM_UID:$SLURM_GID "$GALAXY_VENV" && \
    virtualenv "$GALAXY_VENV" && \
    . "$GALAXY_VENV"/bin/activate && \
    pip install --upgrade pip && \
    pip install galaxy-lib && \
    pip install -r "$GALAXY_VENV"/requirements.txt --index-url https://wheels.galaxyproject.org/simple && \
    rm -rf ~/.cache/

VOLUME ["/export/", "/var/lib/docker"]
CMD ["/usr/bin/startup.sh"]
