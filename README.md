Setup
====
These are the necessary command to create a scalable swarm cluster and demo some of the features of docker.

Setup service discovery box
---------------------------------
- docker-machine create --driver virtualbox consul

- eval $(docker-machine env consul)

- export CONSUL_IP=$(docker-machine ip consul)

- docker run -d -p 8500:8500 -h consul --restart always progrium/consul -server -bootstrap

Setup Swarm cluster
-------------------------
- docker-machine create -d virtualbox  --swarm --swarm-master --swarm-discovery="consul://$(docker-machine ip consul):8500"  --engine-opt="cluster-store=consul://$(docker-machine ip consul):8500" --engine-opt="cluster-advertise=eth1:2376" swarm-master

- docker-machine create -d virtualbox  --swarm --swarm-discovery="consul://$(docker-machine ip consul):8500"  --engine-opt="cluster-store=consul://$(docker-machine ip consul):8500" --engine-opt="cluster-advertise=eth1:2376" swarm1

- docker-machine create -d virtualbox  --swarm --swarm-discovery="consul://$(docker-machine ip consul):8500"  --engine-opt="cluster-store=consul://$(docker-machine ip consul):8500" --engine-opt="cluster-advertise=eth1:2376" swarm2

Add registrator to each swarm box
------------------------------------
- eval $(docker-machine env swarm-master)

- docker run -d --name=registrator -h $(docker-machine ip swarm-master)  --volume=/var/run/docker.sock:/tmp/docker.sock gliderlabs/registrator:latest consul://$(docker-machine ip consul):8500/

- eval $(docker-machine env swarm1)

- docker run -d --name=registrator -h $(docker-machine ip swarm1) --volume=/var/run/docker.sock:/tmp/docker.sock gliderlabs/registrator:latest consul://$(docker-machine ip consul):8500/

- eval $(docker-machine env swarm2)

- docker run -d --name=registrator -h $(docker-machine ip swarm2) --volume=/var/run/docker.sock:/tmp/docker.sock gliderlabs/registrator:latest consul://$(docker-machine ip consul):8500/

Test
-------
- eval $(docker-machine env -swarm swarm-master)

- docker-compose up -d

- docker-compose scale web=3

- docker run -itd --name=web --net=consulswarmdemo_front-tier --env="constraint:node==swarm2" -p 80:80 nginx

- Browser: http://192.168.99.101/

- Browser: http://192.168.99.100:8500/ui/#/dc1/kv/docker/swarm/nodes/

- Command line: curl http://192.168.99.100:8500/v1/kv/docker/swarm/nodes/\?recurse
