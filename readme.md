# Pre-requisites
Before we get started installing the Prometheus/Grafana stack, ensure you install the latest version of [docker](https://docs.docker.com/engine/install/) on your Docker host machine.

# Installation & Configuration
Clone the project locally to your Docker host.

Before you start, you need to change ODP Connect host and port values in Prometheus configuration. Edit the `./config/prometheus/prometheus.yml` file. The `targets` section is where you define what should be monitored by Prometheus. 

Once configurations are done let's start it up. From the repository root directory run the following command:
```sh
docker-compose up -d
```
At that stage Prometheus should be accesible via `http://<Host IP Address>:9090`, for example http://localhost:9090. To make sure that data is being pulled from ODP Connect endpoint, navigate to `Status` -> `Targets` and check the state of the endpoint. It must be `UP`. If it is not, stop the stack by executing the following command:
```shell
docker-compose down
```
check the `prometheus.yaml` file once again and make sure url and port values are correct.

The Grafana Dashboard should be accessible via: `http://<Host IP Address>:3000/dashboards` for example http://localhost:3000/dashboards (default credentials are `admin / admin`).