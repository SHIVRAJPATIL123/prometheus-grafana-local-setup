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
