# EII distributed services centralized logging using ELK

"ELK" is the acronym for three open source projects: Elasticsearch, Logstash, and Kibana. Elasticsearch is a search and analytics engine. Logstash is a server‑side data processing pipeline that ingests data from multiple sources simultaneously, transforms it, and then sends it to a "stash" like Elasticsearch. Kibana lets users visualize data with charts and graphs in Elasticsearch. 

The guide below demonstrates how the logs of EII services running in distributed environment can be centralized via ELK stack. It allows you to search all the logs in a single place.

## Pre-requisites

1.  EII centralized logging using ELK expects a set of config, interfaces & public private keys to be present in ETCD as a pre-requisite.
    To achieve this, please ensure an entry for `remote_logging` with its relative path from [IEdgeInsights](../) directory is set in the use-case yml file present in [build/usecases](https://github.com/open-edge-insights/eii-core/tree/master/build/usecases) directory. An example has been provided below:
    ```sh
        AppContexts:
        - Grafana
        - InfluxDBConnector
        - Kapacitor
        - Telegraf
        - build/remote_logging
    ```

2. With the above pre-requisite done, please run the below command:
    ```sh
        python3 builder.py -f ./usecases/<use-case>.yml
    ```


3. Generate the certificates required to run the Kibana Server using the following command
    ```
    $ ./generate_kibana_certs.sh test-server-ip
    ```

4. The EII centralized logging architecture can be visualized as eii-containers--->rsyslog--->logstash--->elasticsearch--->kibana

5. The eii-containers logs has to be forwarded to the rsyslog, running onto the local system. 
   For that, the eii-container's logging driver need to be syslog. 
   Anyone wants to forward the eii-container's log to the local rsyslog has to modify the file named '/etc/docker/daemon.json'.
   The sample 'daemon.json' is availale in the directory.
   After the modification of 'daemon.json' docker daemon has to be restarted using the below command

   ```sh
   $ sudo systemctl restart docker

   ```
   In case of sending the logs over a secure channel from docker daemon to rsyslog, please refer the 'daemon.json.secure'.
   In this sample configuration, please replace 
   CA_CERT_PATH : CA certificate path
   DOCKER_CERT_PATH : Docker certificate path
   DOCKER_KEY_PATH : Docker key path

6. The rsyslog has to forward the received logs from containers to logstash.
   The file named 'eii.conf' has to be copied into the directory name '/etc/rsyslog.d/'
   After copying this file, install rsyslog module using the below command(The rsyslog module
   installation is one time activity).
   ```sh
   $ sudo apt-get install rsyslog-gnutls

   ```

   Then rsyslog service need to restarted using the below command
   ```sh
   $ sudo systemctl restart rsyslog

   ```
   For more information on the rsyslog,please refer [https://www.rsyslog.com/plugins/](https://www.rsyslog.com/plugins/)

   In case of receiving the logs from docker over the secure channel, please refer the sample config file 'eii.conf.secure'
   In this sample configuration, please replace 
   CA_CERT_PATH : CA certificate path
   DOCKER_CERT_PATH : Docker certificate path
   DOCKER_KEY_PATH : Docker key path
   If case of docker daemon sending logs to remote rsyslog agent, then replace '127.0.0.1' (address="127.0.0.1") with the
   actual IP address of the machine where rsyslog is running.

7. To start ELK containers Please follow below commands
   ```sh
   $ sudo sysctl -w vm.max_map_count=262144

   ```
8. Refer [README.md](https://github.com/open-edge-insights/eii-core/blob/master/README.md) to provision and run the services

   ```sh
   $ docker-compose -f docker-compose-build.yml build 
   $ docker-compose -f docker-compose.yml up -d # this will work in both DEV or PROD mode automatically

   ```

   Please visit [https://host-ip:5601](https://host-ip:5601) for viewing the logs in KIBANA UI.
   Create new index pattern with the elasticsearch indices`logstash` pattern
   
  > **NOTE**: The certificate attached to kibana is self signed and has to be accepted in
  > browser as an exception. The attached certificates are sample certificates only and need to
  > be replaced for production environment.
  > host-ip : Actual IP address of the machine where kibana is running

9. After above 4 steps, please start/restart the EII services.
   Note:
  > **NOTE**: In case of logs are not showing in the KIBANA UI, restart the rsyslog service
  > using the command `sudo systemctl restart rsyslog`
