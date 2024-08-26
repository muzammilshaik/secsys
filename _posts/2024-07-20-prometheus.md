---
author: [muju]
title: "Prometheus"
description: "A detailed guide on installing and configuring Prometheus on Linux, setting up users, directories, and configuring services for effective monitoring."
categories: ["Linux"]
image: /assets/linux/prometheus/prometheus.png
#toc: false
---

## Introduction

Prometheus is a powerful open-source monitoring system and time series database. This guide walks you through installing and configuring Prometheus on a Linux system. By the end of this tutorial, you'll have Prometheus up and running, ready to monitor your infrastructure.

## 1. Create Prometheus User

First, create a dedicated Prometheus user to enhance security:

```bash
sudo useradd -M prometheus
sudo usermod -L prometheus
```

## 2. Set Up Directories

Next, create the necessary directories for Prometheus and set the correct ownership:

```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

## 3. Download and Extract Prometheus

Download the latest Prometheus release from [GitHub:](https://github.com/prometheus/prometheus/releases){:target="_blank"}

```bash
cd ~
wget https://github.com/prometheus/prometheus/releases/download/prometheus.linux-amd64.tar.gz
sha256sum prometheus.linux-amd64.tar.gz 
# Verify the checksum before extracting
tar xvf prometheus.linux-amd64.tar.gz
```

## 4. Install Prometheus

Copy the Prometheus binaries to the appropriate directories:

```bash
sudo cp prometheus.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus.linux-amd64/promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

Move configuration files to `/etc/prometheus`:

```bash
sudo cp -r prometheus.linux-amd64/prometheus.yml /etc/prometheus/
sudo cp -r prometheus.linux-amd64/consoles /etc/prometheus
sudo cp -r prometheus.linux-amd64/console_libraries /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/prometheus.yml
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

## 5. Configure Prometheus

Edit the Prometheus configuration file:

```bash
sudo vi /etc/prometheus/prometheus.yml
```

```yaml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:port

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:3004']

#################################################################################################

  - job_name: '$NAME'

    metrics_path: '/api/v1/allmetrics'
    params:
      # format: prometheus | prometheus_all_hosts
      # You can use `prometheus_all_hosts` if you want Prometheus to set the `instance` to your hostname instead of IP
      format: [prometheus_all_hosts]
      #
      # source: as-collected | raw | average | sum | volume
      # default is: average
      #source: [as-collected]
      #
      # server name for this prometheus - the default is the client IP
      # for Netdata to uniquely identify it
      server: ['grove']

    honor_labels: true
    static_configs:
      - targets: ['$IP:PORT']

#################################################################################################
                                                                                                                                          
```

This setup configures Prometheus to scrape metrics from itself at `localhost:3004`.

## 6. Create an Init Script for Prometheus

Create the init script to manage the Prometheus service:

```bash
sudo nano /etc/init.d/prometheus
```

Copy the init script from the provided code, then save and close the file.

```bash
#!/bin/sh

### BEGIN INIT INFO
# Provides:          prometheus
# Required-Start:    $remote_fs
# Required-Stop:     $remote_fs
# Should-Start:      $all
# Should-Stop:       $all
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: monitoring system and time series database.
# Description:       Prometheus is a systems and service monitoring system.
#                    It collects metrics from configured targets at given intervals,
#                    evaluates rule expressions, displays the results,
#                    and can trigger alerts if some condition is observed to be true.
### END INIT INFO

set -e

. /lib/lsb/init-functions


NAME=prometheus
DESC="Prometheus monitoring system"
DAEMON=/usr/local/bin/prometheus
USER=prometheus
CONFIGDIR=/etc/prometheus
DATADIR=/var/lib/prometheus/data
PID="/var/run/prometheus/$NAME.pid"
LOG="/var/log/prometheus/$NAME.log"
GOSU=/usr/sbin/gosu

ALERTMANAGER_OPTS=

DAEMON_OPTS="$ALERTMANAGER_OPTS"
DAEMON_OPTS="$DAEMON_OPTS --config.file=$CONFIGDIR/prometheus.yml --storage.tsdb.path=$DATADIR"
DAEMON_OPTS="$DAEMON_OPTS --web.console.templates=$CONFIGDIR/consoles --web.console.libraries=$CONFIGDIR/console_libraries"

# Check if DAEMON binary exist
[ -f $DAEMON ] || exit 0

[ -f "/etc/default/$NAME" ] && . /etc/default/$NAME


service_not_configured () {
  if [ "$1" != "stop" ]; then
    printf "\tPlease configure $NAME and then edit /etc/default/$NAME\n"
    printf "\tand set the \"START\" variable to \"yes\" in order to allow\n"
    printf "\t$NAME to start.\n"
  fi
  exit 0
}

service_checks () {
  # Check if START variable is set to "yes", if not we exit.
  if [ "$START" != "yes" ]; then
    service_not_configured $1
  fi

  # Prepare directories
  mkdir -p "/var/run/prometheus" "/var/log/prometheus"
  chown -R $USER "/var/run/prometheus" "/var/log/prometheus"

  # Check if PID exists
  if [ -f "$PID" ]; then
    PID_NUMBER=`cat $PID`
    if [ -z "`ps axf | grep ${PID_NUMBER} | grep -v grep`" ]; then
      echo "Service was aborted abnormally; clean the PID file and continue..."
      rm -f "$PID"
    else
      echo "Service already started; skip..."
      exit 1
    fi
  fi
}

start () {
  service_checks $1
  $GOSU $USER   $DAEMON $DAEMON_OPTS  > $LOG 2>&1  &
  RETVAL=$?
  echo $! > $PID

  if [ $RETVAL ]; then
    log_end_msg 0
  else
    log_end_msg 1
  fi
}

stop () {
  if start-stop-daemon --retry TERM/5/KILL/5 --oknodo --stop --quiet --pidfile $PID 2>&1 1>$LOG
  then
    log_end_msg 0
    rm $PID
  else
    log_end_msg 1
  fi
}


case "$1" in
  start)
    log_daemon_msg "Starting $DESC -" "$NAME"
    start
    ;;

  stop)
    log_daemon_msg "Stopping $DESC -" "$NAME"
    stop
    ;;

  reload)
    log_daemon_msg "Reloading $DESC configuration -" "$NAME"
    if start-stop-daemon --stop --signal HUP --quiet --oknodo --pidfile $PID --startas /bin/bash -- -c "exec $DAEMON $DAEMON_OPTS > $LOG 2>&1"
    then
      log_end_msg 0
    else
      log_end_msg 1
      fi
    ;;

  restart|force-reload)
    log_daemon_msg "Restarting $DESC -" "$NAME"
    stop
    start
    ;;

  syntax)
    $DAEMON --help
    ;;

  status)
    status_of_proc -p $PID $DAEMON $NAME
    ;;

  *)
    log_action_msg "Usage: /etc/init.d/$NAME {start|stop|reload|restart|force-reload|syntax|status}"
    ;;
esac

exit 0

```

## 7. Start and Enable Prometheus Service

To start and restart the Prometheus service, use the following commands:

```bash
sudo service prometheus start
sudo service prometheus enable
sudo service prometheus status
```

![prometheus](/assets/linux/prometheus/prometheus1.png){: width="800" height="500" }

For more details and advanced configurations, refer to the official [Prometheus documentation.](https://prometheus.io/){:target="_blank"}