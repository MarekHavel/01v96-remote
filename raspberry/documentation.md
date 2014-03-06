# Setup on Raspberry Pi

The Raspberry Pi is configured to receive the MIDI signal through its serial port input on the GPIOs. The application is run by forever which ensures that the Node.JS server is restarted after an exception occurs.

**Configuration and software installation can be automated by executing the setup.sh script:**

    chmod +x setup.sh
    sudo ./setup.sh


## Hardware and configuration

Follow the instructions on http://www.siliconstuff.com/2012/08/serial-port-midi-on-raspberry-pi.html

## Software

### Prerequisites

-   You have installed **Raspbian** on your Raspberry Pi

### Installation of required software

Install software dependencies:

    sudo apt-get -y install git python build-essential libasound2-dev

Install the Node.JS server:

    sudo mkdir /opt/node
    wget http://nodejs.org/dist/v0.10.24/node-v0.10.24-linux-arm-pi.tar.gz
    tar xvzf node-v0.10.24-linux-arm-pi.tar.gz
    sudo cp -r node-v0.10.24-linux-arm-pi/* /opt/node
    rm -f -r node-v0.10.24-linux-arm-pi
    sudo ln -s /opt/node/bin/node /usr/bin/node
    sudo ln -s /opt/node/bin/npm /usr/bin/npm
    sudo ln -s /opt/node/lib /usr/lib/node

Install forever:

    sudo /opt/node/bin/npm install -g forever


### Project setup

    cd /home/pi
    git clone https://github.com/kryops/01v96-remote.git
    cd 01v96-remote
    npm install

### init script

Create the file */etc/init.d/forever-01v96-remote* with the following content:

    #!/bin/bash

    ### BEGIN INIT INFO
    # Provides: Forever 01v96-remote
    # Required-Start: $remote_fs $syslog
    # Required-Stop: $remote_fs $syslog
    # Default-Start: 2 3 4 5
    # Default-Stop: 0 1 6
    # Short-Description: Forever 01v96-remote Autostart
    # Description: Forever 01v96-remote Autostart
    ### END INIT INFO

    NAME="Forever NodeJS"
    EXE=/usr/bin/forever
    SCRIPT="/home/pi/01v96-remote/server.js serialport"
    USER=pi
    OUT=/var/log/01v96-remote/forever.log

    if [ "$(whoami)" != "root" ]; then
        echo "This script must be run with root privileges!"
        echo "Try sudo $0"
        exit 1
    fi

    case "$1" in

    start)
        echo "starting $NAME: $EXE $PARAM"
        sudo -u $USER $EXE start -a -l $OUT $SCRIPT
        ;;

    stop)
        echo "stopping $NAME"
        sudo -u $USER $EXE stop $SCRIPT
        ;;

    restart)
        $0 stop
        $0 start
        ;;

    *)
        echo "usage: $0 (start|stop|restart)"
    esac

    exit 0

Activate it:

    chmod 755 /etc/init.d/forever-01v96-remote
    update-rc.d forever-01v96-remote defaults


To finish the installation, reboot your Raspberry Pi.


## Usage, Maintenance

Starting and stopping the application:

    /etc/init.d/forever-01v96-remote start
    /etc/init.d/forever-01v96-remote stop
    /etc/init.d/forever-01v96-remote restart

The application log is written to */var/log/01v96-remote/forever.log*