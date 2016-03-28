Setup
---------
docker-machine create --driver virtualbox consul

docker run -d -p "8500:8500" -h "consul" progrium/consul -server -bootstrap

docker-machine create --driver virtualbox --swarm --swarm-master --swarm-discovery consul://192.168.99.100:8500/ swarm-master

docker-machine create --driver virtualbox --swarm --swarm-discovery consul://192.168.99.100:8500/ swarm1

docker-machine create --driver virtualbox --swarm --swarm-discovery consul://192.168.99.100:8500/ swarm2

docker run -d -e constraint:node==swarm-master --net=host --volume=/var/run/docker.sock:/tmp/docker.sock gliderlabs/registrator:latest consul://192.168.99.100:8500/

docker run -d -e constraint:node==swarm1 --net=host --volume=/var/run/docker.sock:/tmp/docker.sock gliderlabs/registrator:latest consul://192.168.99.100:8500/

docker run -d -e constraint:node==swarm2 --net=host --volume=/var/run/docker.sock:/tmp/docker.sock gliderlabs/registrator:latest consul://192.168.99.100:8500/


Test
-------
Browser:           http://192.168.99.100:8500/ui/#/dc1/kv/docker/swarm/nodes/
Command line: curl http://192.168.99.101:8500/v1/kv/docker/swarm/nodes/\?recurse
