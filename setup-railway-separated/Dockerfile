# Methodology fork 2026-10-17 (Work OS): pin the base to the exact running versions (frappe 16.30.0 / erpnext 16.31.1)
# so a rebuild never upgrades the framework, and add Frappe HR (hrms v16.16.0, Aug 7 2026, frappe >=16.0 <17).
FROM thspacecode/erpnext-docker:16-F30.0_E31.1

USER root

RUN apt-get update \
    && apt-get install -y -q \
    # [fix] "debconf: delaying package configuration, since apt-utils is not installed"
    apt-utils \
    # [fix] "debconf: unable to initialize frontend: Dialog"
    # https://github.com/moby/moby/issues/27988
    && echo "debconf debconf/frontend select Noninteractive" | debconf-set-selections \
    ###############################################
    # add sudo
    ###############################################
    && apt-get install -y -q sudo \
    && echo "frappe ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers \
    ###############################################
    # add supervisor
    ###############################################
    && apt-get install -y -q \
    supervisor \
    nginx \
    ###############################################
    # clean-up
    ###############################################
    && apt-get autoremove --purge -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*

# copy built_sites from sites (hrms is fetched+built in the SAME RUN step, in case sites/ is a VOLUME in the base image)
RUN su frappe -c "cd /home/frappe/frappe-bench && bench get-app --branch v16.16.0 hrms https://github.com/frappe/hrms" \
    && { grep -qx hrms /home/frappe/frappe-bench/sites/apps.txt || echo hrms >> /home/frappe/frappe-bench/sites/apps.txt; } \
    && test -e /home/frappe/frappe-bench/sites/assets/hrms \
    && mkdir -p /home/frappe/frappe-bench/built_sites \
    && cp -r /home/frappe/frappe-bench/sites/assets /home/frappe/frappe-bench/built_sites/assets \
    && cp /home/frappe/frappe-bench/sites/apps.json /home/frappe/frappe-bench/built_sites/apps.json \
    && cp /home/frappe/frappe-bench/sites/apps.txt /home/frappe/frappe-bench/built_sites/apps.txt \
    && chown -R frappe:frappe /home/frappe/frappe-bench/built_sites

# copy template
COPY ./template/temp_nginx.conf /home/frappe/temp_nginx.conf
COPY ./template/temp_supervisor.conf /home/frappe/temp_supervisor.conf

# copy script
COPY --chown=frappe --chmod=0755 ./script/railway-setup.sh /home/frappe/frappe-bench/railway-setup.sh
COPY --chown=frappe --chmod=0755 ./script/railway-entrypoint.sh /usr/local/bin/railway-entrypoint.sh
COPY --chown=frappe --chmod=0755 ./script/railway-cmd.sh /usr/local/bin/railway-cmd.sh

USER root

ENTRYPOINT ["/usr/local/bin/railway-entrypoint.sh"]
CMD ["/usr/local/bin/railway-cmd.sh"]

EXPOSE 80
