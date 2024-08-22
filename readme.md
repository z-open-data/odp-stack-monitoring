The main purpose of this repository is to provide you with sample Grafana dashboard to monitor you ODP stack. Dashboard is focusing on main metrics that allow you to verify that your ODP stack is running properly and is in good state.

# Enable metrics for Data Broker and Data Connect

## Data Connect
In order to enable Data Connect metrics, you need to edit config file (`connect.yaml`).

First of all you need to make sure that your Data Connect config contains these entries:
```yaml
server:
  address: 0.0.0.0
  port: 9070
```
This will enable Spring Boot server and allow you to publish Prometheus endpoint with metrics.
If port 9070 is not available, it can be changed to other available value.

Next you need to add these entries:
```yaml
management:
  endpoints:
    web:
      exposure:
        include: prometheus
```
This will enable Prometheus endpoint with Data Connect, JVM and other metrics. Once enabled, metrics will be available via: `http://[HOST]:[PORT]/actuator/prometheus` where:
- `HOST` - `server.address` value from `config.yaml`
- `PORT` - `server.port` value from `config.yaml`
In our case it will be `http://localhost:9070/actuator/prometheus`.

Data Connect metrics always start with `odp_server_*`.

## Data Broker
To enable Data Broker metrics, we need to modify `PARMLIB` member and add these parameters:
```                                  
KAY.CIDB.STATS=ON
```
This will enable stats subsystem. 

By default, data is being sent every 30 seconds. To change that, you can add an entry to overwrite the default value. 

The following parameter will set refresh interval to 60 seconds:
```
KAY.CIDB.STATS.INTERVAL=60
```

The following entries will create a forwarder and point stats data to your Data Connect instance defined by `SINK_HOST` and `SINK_PORT` parameters.
```
KAY.CIDB.FWD.ST.SOURCE_STORE=STATS                      
KAY.CIDB.FWD.ST.SINK_HOST=[DATA_CONNECT_HOST]   
KAY.CIDB.FWD.ST.SINK_PORT=[DATA_CONNECT_PORT]
```
Once all that is done, restart Data Broker for new settings to take effect.

At that point, both Data Connect and Data Broker metrics (`odp_server_*` and `odp_broker_*`) will be available on your Prometheus endpoint.

# Installation & Configuration
There are two ways of how you can use provided dashboard:
- in existing Prometheus/Grafana environment
- in dedicated Docker based environment

## Existing Prometheus/Grafana setup
If there is an existing Prometheus/Grafana setup running, provided dashboard can be imported with some minor adjustments.
- Prometheus needs to be configured to pull data from new endpoint.
  Edit the `prometheus.yml` file and add new scrape job to your `scrape_configs`:
  ```yaml
    scrape_configs:
      - job_name: 'odp-connect-stats'
        scrape_interval: 15s
        static_configs:
          - targets: ['ODP_ENDPOINT_IP:PORT']
        metrics_path: '/actuator/prometheus'
  ```
  Where:
    - `ODP_ENDPOINT_IP` - ip address of an OMEGAMON Data Connect instance
    - `PORT` - `server.port` value from `config.yaml`
  
  Restart Prometheus after changes.


- Now you can import dashboard to your Grafana instance. Navigate to `Dashboards` section and click on the blue `New` button in the right upper corner, select `Import` option. Below the section `Find and import dashboards for common applications at grafana.com/dashboards` enter dashboard ID for this dashboard - `21708`. Click `Load` button.
In the lower section of the screen select appropriate Prometheus datasource (the one you updated a moment ago) and click `Import`.
  You should be able to see metrics of your ODP Stack in newly added dashboard.

## Dedicated Docker setup
Before you get started installing the Prometheus/Grafana stack, ensure you install the latest version of [docker](https://docs.docker.com/engine/install/) on your Docker host machine.

Clone the project repository.

You need to change OMEGAMON Data Connect host and port values in Prometheus configuration. Edit the `<repository root folder>/config/prometheus/prometheus.yml` file. The `targets` section is where you define what should be monitored by Prometheus.
```yaml
    scrape_configs:
      - job_name: 'odp-connect-stats'
        scrape_interval: 15s
        static_configs:
          - targets: ['ODP_ENDPOINT_IP:PORT']
        metrics_path: '/actuator/prometheus'
  ```
Where:
- `ODP_ENDPOINT_IP` - ip address of an OMEGAMON Data Connect instance
- `PORT` - `server.port` value from `config.yaml`

Once configurations are done you can start up Prometheus/Grafana stack. From the repository root directory run the following command:
```sh
docker-compose up -d
```
At that stage Prometheus should be accesible via `http://<Host IP Address>:9090`, for example http://localhost:9090. To make sure that data is being pulled from OMEGAMON Data Connect endpoint, navigate to `Status` -> `Targets` and check the state of the endpoint. It must be `UP`. If it is not, check the `prometheus.yaml` file once again and make sure url and port values are correct. Then restart Prometheus:
```shell
docker-compose restart prometheus
```

The Grafana Dashboard should be accessible via: `http://<Host IP Address>:3000/dashboards` for example http://localhost:3000/dashboards (default credentials are `admin / admin`).