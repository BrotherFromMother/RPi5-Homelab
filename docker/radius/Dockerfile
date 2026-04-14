FROM debian:bookworm-slim

RUN apt-get update && apt-get install -y \
    freeradius \
    freeradius-ldap \
    freeradius-mysql \
    winbind \
    samba-common-bin \
    samba-client \
    make \
    && rm -rf /var/lib/apt/lists/*

RUN ln -sf /etc/freeradius/3.0/mods-available/pap /etc/freeradius/3.0/mods-enabled/pap && \
    ln -sf /etc/freeradius/3.0/mods-available/mschap /etc/freeradius/3.0/mods-enabled/mschap && \
    ln -sf /etc/freeradius/3.0/mods-available/sql /etc/freeradius/3.0/mods-enabled/sql && \
    make -C /etc/freeradius/3.0/certs/
RUN usermod -a -G root freerad
RUN chown -R freerad:freerad /etc/freeradius/3.0/

EXPOSE 1812/udp 1813/udp

CMD ["freeradius", "-f", "-X"]
