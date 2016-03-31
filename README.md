Setup
==

Setup service discovery box
---------------------------------
- docker-machine create --driver virtualbox consul

- eval $(docker-machine env consul)

- export CONSUL_IP=$(docker-machine ssh consul 'ifconfig eth1 | grep "inet addr:" | cut -d: -f2 | cut -d" " -f1')

- docker run -d -p 8500:8500 -h consul --restart always progrium/consul -server -bootstrap

Setup Swarm cluster
-------------------------
- docker-machine create -d virtualbox  --swarm --swarm-master --swarm-discovery="consul://$(docker-machine ip consul):8500"  --engine-opt="cluster-store=consul://$(docker-machine ip consul):8500" --engine-opt="cluster-advertise=eth1:2376" swarm-master

- docker-machine create -d virtualbox  --swarm --swarm-discovery="consul://$(docker-machine ip consul):8500"  --engine-opt="cluster-store=consul://$(docker-machine ip consul):8500" --engine-opt="cluster-advertise=eth1:2376" swarm1

- docker-machine create -d virtualbox  --swarm --swarm-discovery="consul://$(docker-machine ip consul):8500"  --engine-opt="cluster-store=consul://$(docker-machine ip consul):8500" --engine-opt="cluster-advertise=eth1:2376" swarm2

- eval $(docker-machine env -swarm swarm-master)

Add registrator to each swarm box
------------------------------------

- docker run -d --name=registrator -e constraint:node==swarm-master --volume=/var/run/docker.sock:/tmp/docker.sock gliderlabs/registrator:latest consul://$(docker-machine ip consul):8500/

- docker run -d --name=registrator -e constraint:node==swarm1 --volume=/var/run/docker.sock:/tmp/docker.sock gliderlabs/registrator:latest consul://$(docker-machine ip consul):8500/

- docker run -d --name=registrator -e constraint:node==swarm2 --volume=/var/run/docker.sock:/tmp/docker.sock gliderlabs/registrator:latest consul://$(docker-machine ip consul):8500/

Test
-------
- Browser:           http://192.168.99.100:8500/ui/#/dc1/kv/docker/swarm/nodes/
- Command line: curl http://192.168.99.101:8500/v1/kv/docker/swarm/nodes/\?recurse
