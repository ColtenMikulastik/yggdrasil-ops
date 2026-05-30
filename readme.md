
# Yggdrasil: my homelab docker/container environment
- thanks to @SoriPhoono for putting me on to this.

# Logical Choices:
- main backbone for operations:
    - docker (somewhat obvious)
    - portainer: gui management of docker
    - pihole: DNS resolving in local network
    - nginx: reverse proxy (used only locally)

- can only access resources from VPN or if locally connected to home network to increase security, deny by default

- location of ports is based on category of service: Backbone => (80..), Media => (82..), Tools => (83..)

# Table of current services here
| service | port number |
| ------- | ----------- |
| start-Backbone | 80.. |
| portainer | 8001, 8005 |
| pihole | 8002 |
| nginx | 8003 | 
| elasticsearch | 9200 (attempted 8006) |
| logstash | 8007, 8008 GELF |
| kibana | 8009 | 
| start-Media | 81.. |
| audiobookshelf | 8101 |
| start-tools | 82.. |
| karakeep | 8201 |
| openwebui | 8202 |
| homebox | 8203 | 
| nextcloud | 8204 |
| heimdall | 8205, 8206 |
| n8n | 8207 |
- this is of course not to say this is how the services are accessed but just how they are managed

# Resolving hostnames:
- I have a domain: mikulastik.live, by using the wildcard cert I'm able to use nginx to create a TLS layer for all services.
- this means that devices brought into the network can access resources via:
    - heimdall.mikulastik.live: which is a network launch page
    - then all resources can be accessed by hostnames, resolved by the pihole to nginx and then forwarded through a encrypted tunnel to the service.

# Logs and ElasticSearch
- agent logs, like metrics and other information about a host can be taken in by elastic search instance
- Docker logs, can be forwarded if following rules in logger docker container example

# using Github
- I'm not super familar with the word gitops, but it seems to apply here: https://about.gitlab.com/topics/gitops/
- Using github for a declairative description of how the services is to run, and with added bonus of having version control.
    - also I get to share it with people :^)!

# using Infiscal
- for each compose file, there is also a test environment file, this should describe how you should fill the environment contents
- secrets will not be filled, you will have to fill these, and manage them somehow on your own
    - I'm going to use infiscal, through the cloud for now. This increases redundancy, and is a free with some limitations.

