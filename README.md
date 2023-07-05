# prometheus-grafana-local-setup
1)create a directory name nginx 
2)create nginx.conf file inside it 
server {
    listen 80 ;
    server_name localhost;

        location / {
            root /usr/share/nginx/html;
            index index.html;
        }

    location /metrics {
        stub_status on;
        access_log off;
        allow 0.0.0.0;
    }
}
.............
3)also create index.html with your data in it 
4)create a docker file vi Dockerfile 
FROM nginx:latest

RUN rm /etc/nginx/conf.d/default.conf && rm /usr/share/nginx/html/index.html

COPY nginx.conf /etc/nginx/conf.d/

COPY index.html /usr/share/nginx/html/
EXPOSE 90
.............
5)build image using docker build -t nginximg .
6)create nginx container docker run -d -p --name nginxcont 90:80 nginximg   .....container 80port is bind to local 90
7)check its working with localhost:90..index file content   and        localhost:90/metrics show metrics  
8)Now for craeting nodeexporter container for export metrix from nginx to prometheus .
9)docker run -d -p 9113:9113 nginx/nginx-prometheus-exporter:0.10.0 -nginx.scrape-uri=http://192.168.0.61:90/metrics
                                                                        ip conf 3no             device ip port and path of nginxcont
10)check this using localhost:9113/       .... nginx exporter and in metrics we get metrics of nginxcont
..........
11)Now we have to craete prometheus container and mount prometheus.yaml file to it 
12)create directory mkdir prometheus 
13)then create vi prometheus.yaml 
........
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['192.168.0.61:9113']                ip and port of node exporter 

  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']                   ip and port of prometheus container 
....................
14)then create an prometheus conatiner and bound these file by creating volume to promrtheus conatiner default path using command 
15docker run -d -p 9090:9090 -v /home/deq/Documents/monitoring/prometheus/prometheus.yaml:/etc/prometheus/prometheus.yaml --name prometheus prom/prometheus:latest            ....localpath                                                ... container path 
16)check the running using localhost:9090     prometheus appear
17)Now we have to use another dashboard for prometheus so we use Grafana
18)docker run -d -p 3000:3000 --name grafana grafana/grafana  .........create grafana container
19) check using localhost:3000 ...then login using user=admin pass=admin craete your own pass enter grafana dashboard
20) --- enter into connections Add new connection ..prometheus ..create a prometheus data source
21)name ,server url  http://192.168.0.61:9090 auth save and test green   then create dashboard and run queries set alerts 

