Octograph
---------

Python tool for downloading energy consumption data from the
`Octopus Energy API`_ and loading it into `InfluxDB`_ running on Raspberry Pi Type B.

If you think you'd find this useful, but haven't switched to Octopus yet, then
you can follow my referrer link `<https://share.octopus.energy/cyan-sky-371>`_
and you'll receive a £50 bill credit, and so will I :)

In the process, additional metrics will be generated and stored for unit rates
and costs as configured by the user. Suitable for two-rate electricity tariffs
like `Octopus Energy Go`_. Single rate gas readings are also retrieved and
stored.

The secondary unit rate is specified with start and end times, and a timezone
which is useful for the Go tariff where the discount rate changes with
daylight savings time.

Included is an example `Grafana`_ dashboard to visualise the captured data.

.. image:: grafana-dashboard.png
   :width: 800

Installation
============

Tested on Raspberry Pi Type B and Python 3.6 running on Raspberry Pi OS 32bit.

Install python pip using apt:

.. code:: bash

   sudo apt-get install python3-pip
   
Install Grafana:

Raspberry Pi 1 & Raspberry Pi Zero

.. code:: bash

   cd /tmp
   wget https://dl.grafana.com/oss/release/grafana-rpi_6.6.1_armhf.deb
   sudo dpkg -i grafana-rpi_6.6.1_armhf.deb
   apt --fix-broken install

Raspberry Pi 2 and later

.. code:: bash

   cd /tmp
   wget https://dl.grafana.com/oss/release/grafana_6.6.1_armhf.deb
   sudo dpkg -i grafana_6.6.1_armhf.deb
   apt --fix-broken install
   
Enable Grafana at startup and start Grafana:

.. code:: bash
   
   sudo systemctl enable grafana-server
   sudo systemctl start grafana-server
   
Install InfluxDB:

.. code:: bash

   wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
   echo "deb https://repos.influxdata.com/debian buster stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
   sudo apt update
   sudo apt install influxdb
   sudo systemctl unmask influxdb
   sudo systemctl enable influxdb
   sudo systemctl start influxdb

Download Octograph:

.. code:: bash
  
  cd /home/pi
  wget https://github.com/taitadam-eng/octograph/archive/master.zip
  unzip master.zip
  rm master.zip
  

Install the Python requirements with ``pip3``

.. code:: bash

    pip3 install -r octograph-master/app/requirements.txt
    
Fix Pendulum and TZLocal compatability issue:

.. code:: bash

   sudo pip3 uninstall tzlocal
   sudo pip3 install tzlocal==1.5.1
   
Copy the example config and enter required 

.. code:: bash
   
   cp octograph-master/example-octograph.ini octograph-master/octograph.ini
   nano octograph-master/octograph.ini

Usage
=====

Create a configuration file ``octograph.ini`` customised with your Octopus
API key, meter details and energy rate information. This file should be in the
working directory where you run the ``octopus_to_influxdb.py`` command, or
can be passed as an argument.

.. code:: bash

    python3 octograph-master/app/octopus_to_influxdb.py --help

By default, energy data for the previous day will be collected. Optional from
and to ranges may be specified to retrieve larger datasets. It is anticipated
that the script will be run daily by a cron job.

.. code:: bash

    python3 octograph-master/app/octopus_to_influxdb.py --from-date=2020-05-01
    open http://<RASPBERRY PI IP ADDRESS>:3000

The default login credentials for Grafana are admin/admin, and you will be
prompted to set a new password on first login. You should then proceed to add
InfluxDB as a datasource with URL ``http://localhost:8086`` and database
``energy`` if using the Docker version provided. The dashboard provided can
then be imported to review the data.


.. _Octopus Energy API: https://developer.octopus.energy/docs/api/
.. _Octopus Energy Go: https://octopus.energy/go/
.. _InfluxDB: https://www.influxdata.com/time-series-platform/influxdb/
.. _Grafana: https://grafana.com
