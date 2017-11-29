# cgroups-and-containers

## Step 1: Starting with docker containers

- Installation steps
```
sudo apt-get remove docker docker-engine docker.io
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
apt-cache policy docker-ce
sudo apt-get install -y docker-ce
```  

- Start docker
``` 
sudo systemctl status docker
```

- List of all containers
```
sudo docker ps -a
```

- Pull an image from docker hub
```
sudo docker pull <image-name>
```
```
sudo docker pull ubuntu
```
