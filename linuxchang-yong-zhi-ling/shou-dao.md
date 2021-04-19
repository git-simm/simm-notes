1. docker-images的清理

    sudo docker rmi --force `sudo docker images | grep exchange | awk '{print $3}'`



