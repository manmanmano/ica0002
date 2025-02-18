# INSTALL AND CONFIGURE OUR INFRASTRUCTURE WITH ANSIBLE


### !!! THIS MUST BE DONE ON OUR LOCAL MACHINE BEFORE DATA RESTORATION !!!

        ansible-playbook infra.yaml



# MYSQL DATA RESTORATION


### !!! THE FOLLOWING COMMANDS MUST BE EXECUTED ON MYSQL MASTER MACHINE !!!
The default master machine is manmanmano-2, so be sure to execute these commands there.


Before starting clean everything from the /home/backup/restore directory with the following commands:

    1. sudo su

    2. rm /home/backup/restore/*

    3. exit


Follow these steps in order to restore our lost data:

1) Login into the backup user:

        sudo su backup

2) Restore MySQL data from the backup with the following command:

        duplicity restore --no-encryption rsync://manmanmano@backup.clounsible.mano//home/manmanmano/mysql/ /home/backup/restore/

3) Exit from the backup user:

        exit

4) Login into the root user:

        sudo su

5) Paste the retrieved MySQL dump into the database as root using the following command:

        mysql agama < /home/backup/restore/agama.sql 

Now check on our application and see if the data has been successfully restored.



# INFLUXDB DATA RESTORATION


### !!! THE FOLLOWING COMMANDS MUST BE EXECUTED ON THE MACHINE THAT HOSTS INFLUXDB !!!
InfluxDB is located in manmanmano-3, so be sure to execute these commands there.


Before starting clean everything from the /home/backup/restore directory with the following commands:

    1. sudo su

    2. rm /home/backup/restore/*

    3. exit


Follow these steps in order to restore our lost data:

1) Login into the root user:

        sudo su 

2) Stop the telegraf service with the following command:

        service telegraf stop

3) Run:

        influx -execute 'DROP DATABASE telegraf'

4) Login into the backup user:

        sudo su backup

5) Restore InfluxDB data from the backup with the following command:

        duplicity restore --no-encryption rsync://manmanmano@backup.clounsible.mano//home/manmanmano/influxdb/ /home/backup/restore/

6) Take the telegraf restored dump and put it into the database:

        influxd restore -portable -database telegraf /home/backup/restore/


### !!! THE FOLLOWING COMMAND MUST BE RUN ON OUR LOCAL MACHINE !!!
In case the backup was successful, we can restart the service telegraf by running the playbook:
    
        ansible-playbook infra.yaml

Now check the Syslog dashboard in Grafana and see if our data has been restored.



