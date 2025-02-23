FROM torizon/wayland-base-vivante:2

# Basic set of packages.
RUN apt-get -y update && apt-get install -y --no-install-recommends \
    apt-utils \
    net-tools \
    procps

# Packages required by the GUI:
# - libwayland-dev is used to get all the wayland related libs. It will
#   install headers which are not really needed, but we can live with that.
# - libasound2 is required by a Storyboard plugin.
# - libmtdev1 is required by the mtdev SB plugin.
RUN apt-get install -y --no-install-recommends \
    libwayland-dev \
    libasound2 \
    libts-bin

# We use tini to run the script that starts the GUI and backends as a PID
# different from 1. This is part of ensuring that the app gets the proper
# signals upon closing the container and hardware is correctly de initialized.
RUN apt-get install -y --no-install-recommends tini

# Packages required by backends.
# - dbus: required to shut down the system.
# - libgpiod2 for accessing GPIOs.
# - libsqlite3-0 for the DB.
RUN apt-get install -y --no-install-recommends \
    dbus \
    libgpiod2 \
    libsqlite3-0 \
	imagemagick

RUN apt-mark hold dash && apt-get -y upgrade && apt-mark unhold dash \
    && apt-get clean && apt-get autoremove && rm -rf /var/lib/apt/lists/*

# Copy the crank runtime to it's place.
RUN mkdir -p /opt/crank
COPY --chown=root:root crank /opt/crank/

# Copy the GUI to it's place.
RUN mkdir -p /opt/gui
COPY --chown=root:root gui /opt/gui

# Copy backends to it's place.
RUN mkdir -p /opt/backends/
COPY --chown=root:root backends /opt/backends

# Create the data mounting point
RUN mkdir -p /opt/data

# Copy the start script.
COPY --chown=root:root coaguscan.sh /opt/coaguscan.sh

ENV ACCEPT_FSL_EULA=1

ENTRYPOINT ["tini", "-v", "--", "/opt/coaguscan.sh"]
