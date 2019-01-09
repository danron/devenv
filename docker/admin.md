# Docker

## Remove all mir_mir containers

docker ps -a | awk '$2 == "mir_mir" {print $1}' | xargs docker rm

## Remove all repo = <none> images

docker images | awk '$1 == "<none>" {print $3}' | xargs docker rmi
