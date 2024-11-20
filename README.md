# homelab-docker-compose-stacks
A comprehensive list of the stacks I run in my homelab
\
You can find information about each one below.  
\
DDNS-Cloudflare
DDNS to automatically update Cloudflare DNS entries.

```version: '2'
services:
  cloudflare-ddns:
    image: oznu/cloudflare-ddns:latest
    restart: always
    environment:
      - API_KEY=Hk9O2OLFaLhUeiaR6tNb6mE9mRJlo811x3CR5uXi
      - ZONE=matthewsluberski.com
      - PROXIED=false
      # - PUID=
      # - PGID=
```
\
Dozzle
Dozzle is a log collection tool for Docker containers. It collects all the logs and puts them in one place to easily go through. It also monitors resource usage of each container. You also get the ability to export the logs to influxDB for viewing and storing.\

Below is the docker-compose stack you can run by creating a docker-compose.yml file and running sudo docker-compose up -d. To access logs, go to http://:9999\

```version: "3"
services:
  dozzle:
    container_name: dozzle
    image: amir20/dozzle:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - 9999:8080
    ```
```
\
Homarr
Homarr is a dashboard. That's it, simple.

```version: '3'
#---------------------------------------------------------------------#
#     Homarr - A simple, yet powerful dashboard for your server.     #
#---------------------------------------------------------------------#
services:
  homarr:
    container_name: homarr
    image: ghcr.io/ajnart/homarr:latest
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock # Optional, only if you want docker integration
      - ./homarr/configs:/app/data/configs
      - ./homarr/icons:/app/public/icons
      - ./homarr/data:/data
    ports:
      - '7575:7575'
```
\
HomeBox
Homebox is a self-hosted inventory management system. It tracks software and hardware inventory. I use it for tracking software used for my business.

```version: "3.4"

services:
  homebox:
    image: ghcr.io/hay-kot/homebox:latest
#   image: ghcr.io/hay-kot/homebox:latest-rootless
    container_name: homebox
    restart: always
    environment:
    - HBOX_LOG_LEVEL=info
    - HBOX_LOG_FORMAT=text
    - HBOX_WEB_MAX_UPLOAD_SIZE=10
    volumes:
      - homebox-data:/data/
    ports:
      - 3100:7745

volumes:
   homebox-data:
     driver: local
```
\
Lidarr
Lidarr is a music management arr platform similar to Sonarr or Radarr. It allows you to index music files to download and self-host.

```
---
services:
  lidarr:
    image: lscr.io/linuxserver/lidarr:latest
    container_name: lidarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - /path/to/lidarr/config:/config
      - /path/to/music:/etc/media/music
      - /path/to/downloads:/etc/media/musicdownloads #optional
      - /etc/media/successful:/Downloads/complete
    ports:
      - 8686:8686
    restart: unless-stopped
```
\
Media Aggregation Stack
This is the media Aggregation stack that I run for collecting media.
It's considered a *arr stack due to the names of the programs and containers.
The volume paths need to be edited for your setup.  In my case, I created a folder called media in my /etc folder and added the required folder structure.
Required folder structure is as follows:
Media
--->Successful
--->Unsuccessful
--->Movies
--->TVShows

List of containers
Prowlarr - Indexer manager. Acts as an indexer for the other applications. Points to an online NZB indexer to find releases.
SabNZBD - NZB downloader. Downloads the files that Prowlarr grabs and unpacks them
Radarr - Manages Movie files and sends requests to Prowlarr to get files for, then sends to SabNZBD and moves the files after completed.
Sonarr - Manages TV Show files and sends requests to Prowlarr to get files for, then sends to SabNZBD and moves the files after completed.
Overseer - Request platform for browsing and requesting media for the server to download. Connects to a plex account or to a local account for others to access. Requested media is sent to Radarr or Sonarr to follow the flow.
Doplarr - Discord bot that sends requests to Overseer.\

docker-compose.yml

```version: "2.1"
services:
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - /path/to/data:/config
    ports:
      - 9696:9696
    restart: unless-stopped
  sabnzbd:
    image: lscr.io/linuxserver/sabnzbd:latest
    container_name: sabnzbd
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - /etc/sabnzbd:/config
      - /etc/media/successful:/Downloads/complete
      - /etc/media/unsuccessful:/Downloads/incomplete
    ports:
      - 8080:8080
    restart: unless-stopped
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - /etc/radarr:/config
      - /etc/media/successful:/Downloads/complete
      - /etc/media/movies:/Downloads/Movies
    ports:
      - 7878:7878
    restart: unless-stopped
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - /etc/sonarr:/config
      - /etc/media/successful:/Downloads/complete
      - /etc/media/tvshows:/Downloads/tvshows
    ports:
      - 8989:8989
    restart: unless-stopped
  overseerr:
    image: lscr.io/linuxserver/overseerr:latest
    container_name: overseerr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - /path/to/overseerr/config:/config
      - /etc/media/successful:/Downloads/complete
      - /etc/media/movies:/Downloads/Movies
      - /etc/media/tvshows:/Downloads/tvshows
    ports:
      - 5055:5055
    restart: unless-stopped
  doplarr:
    image: lscr.io/linuxserver/doplarr:latest
    container_name: doplarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - DISCORD__TOKEN=<--->
      - OVERSEERR__API=<--->
      - OVERSEERR__URL=http://10.10.0.4:5055
      - RADARR__API=
      - RADARR__URL=
      - SONARR__API=
      - SONARR__URL=
    #  - DISCORD__MAX_RESULTS=25 #optional
     # - DISCORD__REQUESTED_MSG_STYLE=:plain #optional
     # - SONARR__QUALITY_PROFILE= #optional
     # - RADARR__QUALITY_PROFILE= #optional
     # - SONARR__ROOTFOLDER= #optional
    #  - RADARR__ROOTFOLDER= #optional
     # - SONARR__LANGUAGE_PROFILE= #optional
     # - OVERSEERR__DEFAULT_ID= #optional
      #- PARTIAL_SEASONS=true #optional
      #- LOG_LEVEL=:info #optional
      #- JAVA_OPTS= #optional
    volumes:
      - /path/to/doplarr/config:/config
    restart: unless-stopped
```
\
Monitoring Stack
This is a Grafana, Prometheus, Node_Exporter, CAdvisor stack for Monitoring. This allows for good monitoring for services and processes.

```
version: '3'

volumes:
  prometheus-data:
    driver: local
  grafana-data:
    driver: local

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - /etc/prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    restart: unless-stopped
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    restart: unless-stopped

  node_exporter:
    image: quay.io/prometheus/node-exporter:latest
    container_name: node_exporter
    command:
      - '--path.rootfs=/host'
    pid: host
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'

  cadvisor:
    image: google/cadvisor:latest
    container_name: cadvisor
    # ports:
    #   - "8880:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    devices:
      - /dev/kmsg
    restart: unless-stopped
```
\
Otter Wiki
Otter Wiki is an easy and simple wiki.  Very nice to use
```
version: '3'
services:
  otterwiki:
    image: redimp/otterwiki:2
    restart: unless-stopped
    ports:
      - 8008:80
    volumes:
      - ./app-data:/app-data
```
\
PingVin
Pingvin is a self-hosted Weshare or bit.ly alternative for security.
```
version: '3.8'
services:
  pingvin-share:
    image: stonith404/pingvin-share
    restart: unless-stopped
    ports:
      - 3000:3000
    volumes:
      - "./data:/opt/app/backend/data"
      - "./data/images:/opt/app/frontend/public/img"
```
