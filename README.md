Docker-compose-Nginx-Reverse-Proxy
===================================

#Quickstart

##Change configuration
Here we will add new service newservice to existing domain mydomain.com so we can access it with
newservice.mydomain.com

Add new upstream to nginx.cong

    server {
        listen 80;
        server_name newservice.mydomain.com newservice.mydomain.com_dev;

        location / {
            resolver 127.0.0.11 valid=30s;
            set $upstream      newservice_1:8080;
            proxy_pass         http://$upstream;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
        }
    }
add new network to services.networks and networks in docker-compose.yml

    services:
        reverseproxy:
            image: nginx:alpine
            ports:
                - 80:80
            restart: always
            volumes:
                - ./nginx.conf:/etc/nginx/nginx.conf
            networks:
                - newservice_net
    
    networks:
      new_service_net:
        external:
          name: newservice_default


add dev host to /etc/hosts 

    sudo -- bash -c "echo 127.0.0.1 newservice.mydomain.com_dev >> /etc/hosts"

add new dns CNAME to GCP (grachev - zone name, grachev.space - domain, airflow.proxybroker.grachev.space - new subdomain) 

    gcloud dns --project=valiant-striker-270420 record-sets transaction start --zone=grachev
    gcloud dns --project=valiant-striker-270420 record-sets transaction add grachev.space. --name=airflow.proxybroker.grachev.space. --ttl=300 --type=CNAME --zone=grachev
    gcloud dns --project=valiant-striker-270420 record-sets transaction execute --zone=grachev
    
##(Re)start reverse_proxy
Stop existing container, if container is already running: 

    docker-compose down 
Start container
    
    docker-compose up -d
