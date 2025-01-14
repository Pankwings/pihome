# Monitoring Apartment with Raspberry Pi, Prometheus & Grafana

For quite some time, I had a spare Raspberry Pi lying around in my place. And one weekend I came up with idea to make my apartment "smarter". What I mean by saying "smarter" is tracking some metrics of my surroundings.

I have some experience in working with [Prometheus](https://prometheus.io/) and [Grafana](https://grafana.com/), so I decided to incorporate those tools into my solution. Yes, it does sound like overengineering simple task, you can probably get same results in much simpler way : ).

By deploying this project with all its components, you'll be able to track these metrics:
- Room temperature
- Humidity
- Connected network devices (Removed)

## Hardware components

These are all the component, I used in my project:
- [Raspberry Pi 3 Model B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)
- 16 GB microSD card
- [DHT11 Temperature And Humidity Sensor](https://components101.com/dht11-temperature-sensor)
- Mobile phone charger, for powering Raspberry Pi

## Connecting DHT11 Sensor to Raspberry Pi

I connected Ground pin to the Ground of Raspberry PI, Data Pin to GPIO 4 pin, Vcc pin to 3.3V power supply pin.


## Reading sensor data

For reading DHT11 sensor data and feeding it to Prometheus, I chose [DHT11_Python](https://github.com/szazo/DHT11_Python) library, which is quite unstable, and sometimes does not return valid results, so you might get some gaps in your graphs.
You can browse source code of this project, for more details:
    - `application/temperature.py` & `application/dht11.py` for temperature & humidity readings;
    - `application/webapp.py` for prometheus endpoint.


## Prometheus configuration

To scrape metrics from my Flask API, I've added configuration to `prometheus.yml`:

```yaml
global:
    scrape_interval: 30s
scrape_configs:
    - job_name: 'pihome'
      static_configs:
        - targets: [pihome:5000]
```

## Grafana Configuration

Then, in `/etc/grafana/provisioning`, I've added datasource configuration:
```yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090/
    access: proxy
    isDefault: true
```
It is also possible to add Grafana dashboards to provisioning folder as json files, so that you don't need to create new dashboard each time you re-deploy Grafana.

## Connecting everything together

To make everything portable and easy to install, I packed my Flask API to Docker image and configured all services in `docker-compose.yaml`.
To deploy whole stack you have to add `.env` file with some configuration properties:
```
GRAFANA_PASSWORD=yourpassword
ADMIN_USER="admin"  #for caddy basic auth
ADMIN_PASSWORD_HASH="JDJhJDE0JC82VVpaUk9nR3hMQ2FtOXprdVJ4c2VNUXJhYjh5YWR3bmJOMHR2SUNTSE5ZVVNCU1pXaEFh"  #generated hash password of "admin". Type "admin" in the password if using this hash.
```

After adding `.env`, file run `docker-compose build` & `docker-compose up -d`.

## Result


## Usefull resources
- https://github.com/pdambrauskas/pihome
- https://www.freva.com/2019/05/21/hc-sr501-pir-motion-sensor-on-raspberry-pi/
- https://github.com/szazo/DHT11_Python

