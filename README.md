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

- We can verify the information above in host machine too

```
cd /sys/fs/cgroup/cpuset
ls
```

Output:
```
demy@demy-VirtualBox:/sys/fs/cgroup/cpuset$ ls
cgroup.clone_children  cpuset.cpu_exclusive   cpuset.effective_mems  cpuset.memory_migrate           cpuset.memory_spread_page  cpuset.sched_load_balance        notify_on_release
cgroup.procs           cpuset.cpus            cpuset.mem_exclusive   cpuset.memory_pressure          cpuset.memory_spread_slab  cpuset.sched_relax_domain_level  release_agent
cgroup.sane_behavior   cpuset.effective_cpus  cpuset.mem_hardwall    cpuset.memory_pressure_enabled  cpuset.mems                docker                           tasks

```

- Important Notes: 
    - tasks: we can see every task running on the system (root cgroup)
    - cpuset.cpus: we can see all cpus (root cgroup)

- Find docker cgroup in cpuset hierarchy
```
cd docker
ls
```

Output:
```
(none)
```

```
cd /fe558ff7ba29d59c8ff1708546ad6f1bd07c6cf34a64768b2242ea63478be73f
ls
```

Output:
```
cgroup.clone_children  cpuset.cpus            cpuset.mem_exclusive   cpuset.memory_pressure     cpuset.mems                      notify_on_release
cgroup.procs           cpuset.effective_cpus  cpuset.mem_hardwall    cpuset.memory_spread_page  cpuset.sched_load_balance        tasks
cpuset.cpu_exclusive   cpuset.effective_mems  cpuset.memory_migrate  cpuset.memory_spread_slab  cpuset.sched_relax_domain_level
```

- In file tasks there is only one pid, the PID of the container
```
cat tasks
```

Output:
```
11274
```
Verify it is container's PID by killing it.. The container exits xD

### Tunning cpu affinity
    1. taskset-way `< taskset -cap 11274>`
    2. cgroups-way `< cat cpuset.cpus >`

- Can we change affinity with taskset ?
    - `< taskset >` command will be filtered out by cpuset.cpus and if the new mask is not included in cpuset.cpus then the `< taskset >` will be ignored or fail
    - example#1: `< taskset -cap 2 11274 >` will success and the affinity will change as we can see by '< cat /proc/self/status | grep Cpus_allowed >' inside the container
    - example#2: `< sudo vim cpuset.cpus >` 0-3 -> 2-3 and then `< taskset -cap 0 11274 >` ``` pid 11274's current affinity list: 2
    taskset: failed to set pid 11274's affinity: Invalid argument```
    - example#3: current container cpu affinity let be 2-3 and we  `< taskset -cap 0-2 11274 >`
        the new affinity will be ```pid 11274's current affinity list: 2,3
                                    pid 11274's new affinity list: 2```

