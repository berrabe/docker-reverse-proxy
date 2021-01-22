<p align="center">
  <img src="docs/logo.png">
</p>

<br/><br/>
### TLDR;
---
This docker is used to create a reverse proxy on the http / https protocol, based on nginx.

everything in this repo is based from [nginx-proxy/nginx-proxy](https://github.com/nginx-proxy/nginx-proxy) for nginx-reverse-proxy ... 
and this [docker-letsencrypt-nginx-proxy-companion](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion) for auto generated SSL certs with letsencrypt 

make sure you have installed **docker** and **docker-compose**, if not installed, please see the [official docker documentation here](https://docs.docker.com/get-docker/). And make sure you have setup the domain to the ip server you want to install this reverse-proxy on


<br/><br/>
### USAGE
---
- first step, clone this repo
```sh
> git clone https://github.com/berrabe/docker-reverse-proxy.git
> cd docker-reverse-proxy
```

- then, you have to change some parameters that contain the word `example.com`
```yaml
# 1
        environment:
            - HTTPS_METHOD=noredirect
            # used as the default domain if the domain / ip being accessed does not exist
            - DEFAULT_HOST=example.com

# 2
        environment:
            # used as an email notification if the SSL certs is about to expire (90 days)
            - DEFAULT_EMAIL=xxx@example.com     
```

- run this compose file with
```sh
> docker-compose up -d

# for logging
> docker-compose logs -f
```


<br/><br/>
### TESTING
---

to test whether the reverse proxy is working or not, we have to run the **"dummy"** container which will trigger the docker reverse-proxy to auto-generated config and ssl certs against the domains in env var `VIRTUAL_HOST` and `LETSENCRYPT_HOST`
- change `testing-dummy.example.com` to your domain
```sh
> docker run -d \              
        --name web \
        -e VIRTUAL_HOST=testing-dummy.example.com \
        -e LETSENCRYPT_HOST=testing-dummy.example.com \
        nginx:alpine                        
```

<br/><br/>
### DIFFERENT NETWORK?
---
By default, this docker will detect env var `VIRTUAL_HOST` and `LETSENCRYPT_HOST` on the running container that have same connected network (default is bridge). If you wants to run as docker-compose (which is usually use the auto-generated network). You must setup this **docker-reverse-proxy** to join the auto generated network created by docker-compose automatically through this script

- change `reverse-proxy` to your custom container name (default is reverse-proxy)
```sh
> ./auto_net.brb reverse-proxy
```

- the output display will be like this
```sh
   ./ Docker Auto Join Network \.


 [+] LIST ALL DOCKER NETWORKS 
  |--[-] 329d3a2e32d8 >>> 05nginxdirlist_default
  |--[-] cefc731a02d6 >>> 06monitoringstack_default
  |--[-] a72740956861 >>> berrabe_default
  |--[-] 7f23f3060488 >>> bridge


 [+] LIST [ reverse-proxy ] CONTAINER CONNECTED NETWORKS
  |--[-] 7f23f3060488
  

 [+] DISCONNECT ALL NETWORKS ATTACHED TO [ reverse-proxy ] CONTAINER
  |--[-] Disconecting 7f23f3060488 ... OK


 [+] CONNECTING [ reverse-proxy ] TO ALL NETWORKS
  |--[-] Connecting to 329d3a2e32d8 ... OK
  |--[-] Connecting to cefc731a02d6 ... OK
  |--[-] Connecting to a72740956861 ... OK
  |--[-] Connecting to 7f23f3060488 ... OK


 [+] RESTART [ reverse-proxy ] CONTAINER ... OK
```

- to verify it
```sh
> docker inspect reverse-proxy | grep "NetworkID"

"NetworkID": "329d3a2e32d80d5da8accfc71fdc1142f8ca06d12ae7ed328950136aba6c27c9",
"NetworkID": "cefc731a02d65e46b8085675b978d4abcf60c7a112ef0b829cf137870bc65424",
"NetworkID": "a727409568618db52186fdd2c2ed8bddc36bd2379c098c2b1eabaabf6c08c317",
"NetworkID": "7f23f306048847b82a36cacb517a6121c395ed48a9f3b2f784921915b78501f7",
```

container reverse-proxy which by default runs under a bridge network will automatically connect to all existing networks (except hosts and none).