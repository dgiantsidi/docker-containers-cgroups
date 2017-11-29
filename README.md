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

- Deploy a docker (--rm flag deletes container image after exiting)
```
docker run --rm --name test-cgroups -it ubuntu
```

## Step 2: Find the cgroup attached to each container

- "Inside" the docker we can retrieve cpu affinity and number of processors on which the container is pinned
```
nproc
cat /proc/self/status | grep Cpus_allowed
```

Output: 
```
Cpus_allowed:f
Cpus_allowed_list:0-3
        
```

- We can also find all cgroups this container is attached to
```
cat /proc/self/cpusets
cat /proc/self/cgroups
```

Output:
```
/docker/fe558ff7ba29d59c8ff1708546ad6f1bd07c6cf34a64768b2242ea63478be73f



11:blkio:/docker/fe558ff7ba29d59c8ff1708546ad6f1bd07c6cf34a64768b2242ea63478be73f
10:pids:/docker/fe558ff7ba29d59c8ff1708546ad6f1bd07c6cf34a64768b2242ea63478be73f
9:hugetlb:/docker/fe558ff7ba29d59c8ff1708546ad6f1bd07c6cf34a64768b2242ea63478be73f
8:perf_event:/docker/fe558ff7ba29d59c8ff1708546ad6f1bd07c6cf34a64768b2242ea63478be73f
7:freezer:/docker/fe558ff7ba29d59c8ff1708546ad6f1bd07c6cf34a64768b2242ea63478be73f
6:cpuset:/docker/fe558ff7ba29d59c8ff1708546ad6f1bd07c6cf34a64768b2242ea63478be73f
5:devices:/docker/fe558ff7ba29d59c8ff1708546ad6f1bd07c6cf34a64768b2242ea63478be73f
4:memory:/docker/fe558ff7ba29d59c8ff1708546ad6f1bd07c6cf34a64768b2242ea63478be73f
3:net_cls,net_prio:/docker/fe558ff7ba29d59c8ff1708546ad6f1bd07c6cf34a64768b2242ea63478be73f
2:cpu,cpuacct:/docker/fe558ff7ba29d59c8ff1708546ad6f1bd07c6cf34a64768b2242ea63478be73f
1:name=systemd:/docker/fe558ff7ba29d59c8ff1708546ad6f1bd07c6cf34a64768b2242ea63478be73f

```


