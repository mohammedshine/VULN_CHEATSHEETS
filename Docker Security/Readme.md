Basic Docker Security checklist below.

Docker Penetration Test Checklist


These questions will help us identify if we are inside a container or not. If the answer to any of the following questions is yes, we are most likely running inside a container.

If we know we are inside a container, we can look for vulnerabilities inside the container. If we know we are not running inside a container, we can look for vulnerabilities on the host


• Does /.dockerenv exist? 
Execute “ls /.dockerenv” to see if /.dockerenv exists.

• Does /proc/1/cgroup contain “/docker/”? 
Execute “grep '/docker/' /proc/1/cgroup” to find all lines in /proc/1/cgroup containing “/docker/”.

• Are there fewer than 5 processes? 
Execute “ps aux” to view all processes.

• Is the process with process id 1 a common initial process?
Execute “ps -p1” to view the process with process id 1 and check if it is a common initial process (.e.g. systemd or init).

• Are common libraries and binaries not present on the system? 
We can use the which command to find available binaries. For example, “which sudo” will tell us if the sudo binary is available.

Finding vulnerabilities in Containers

The following questions and steps are meant to identify interesting parts
and weak spots inside containers. 

• What is the current user? 
Execute “id” to see what the current user is and what groups it is in.

• Which users are available on the system? 
Read /etc/passwd to see what users are available.

• What is the operating system of the container? 
Read /etc/os-release to get information about the operating system.

• Which processes are running? 
Execute “ps aux” to view all processes.

• What is the host operating system? 
Execute “uname -a” to get information about the kernel and the underlying host operating system.

• Which capabilities do the processes in the container have?
Get the current capabilities value by running “grep CapEff /proc/self/status” and decode it with “capsh --decode=value”. Capsh can be run on a different system.

• Is the container running in privileged mode?
If the CapEff value of the previous step equals 0000003fffffffff,the container is running in privileged mode and we are able to escape it.

• What volumes are mounted?
Read /proc/mounts to see all mounts including the volumes.

• Is there sensitive information stored in environment variables? 
The “env” command will list all environment variables. We should check these for sensitive information.

• Is the Docker Socket mounted inside the container? 
Check /proc/mounts to see if docker.sock (or some similar named socket) is mounted inside the container. /run/docker.sock is a common mount point. If we find it, we can escape the container and interact with the Docker daemon on the host.

• What hosts are reachable on the network? 
If possible, use nmap to scan the local network for reachable hosts. The IPv4 address of the container can be found in /etc/hosts.

Finding vulnerabilities on the Host

The following questions and steps are meant to identify interesting parts
and weak spots on hosts running Docker. 

• What is the version of Docker? 
Run “docker --version” to find the version of Docker. We will need to check if there are any known software related bugs in this version of Docker. 

• Which CIS Docker Benchmark guidelines are implemented incorrectly or are not being followed? 
Run Docker Bench for Security2 to quickly see which CIS Docker Benchmark guidelines are not being followed.

• Which users are allowed to interact with the Docker socket?
Execute “ls -l /var/run/docker.sock” to see the owner and group of /var/run/docker.sock and which users have read and write access to it. Users that have read and write permissions to the Docker socket are allowed to interact with it.

• Who is in the docker group? 
Check which users are in the group identified in the previous step (by default docker) by executing “grep docker /etc/group”.

• Is the setuid bit set on the Docker client binary? 
Check the permissions (including whether the setuid bit is set) of the Docker binary by executing “ls -l $(which docker)”.

• What images are available? 
List the available images by running “docker images -a”.

• What containers are available? 
List all containers (running and stopped) by running “docker ps -a”.

• How is the Docker daemon started? 
Check configuration files (e.g. /usr/lib/systemd/system/docker.service and /etc/docker/daemon.json) for information on how the Docker daemon is started.

• Do any docker-compose.yaml files exist? 
Find all docker-compose.yaml files using “find / -name "dockercompose.*"”.

• Do any .docker/config.json files exist? 
Read the config.json files in all directories by running “cat /home/*/.docker/config.json”.

• Are the iptables rules set for both the host and the containers?
List the iptables by running “iptables -vnL” and “iptables -t filter -vnL”


Tools

Docker Bench for Security (Docker’s own tool)
Dockscan (checks for misconfigurations)
Aqua Security’s MicroScanner(Docker Image Analysis Tool)
CoreOS/Clair (Docker Image Analysis Tools)
Harpoon(Docker Image Analysis Tools)
Break Out of the Box (Exploitation tool)
Metasploit (Exploitation tool)

Kubernetes Checklist


1. Check your kubernetes server version

2. Check anonymous access
#Kube API 
curl -k https://<master_ip>:<port>
#Check etcdctl (binary on https://github.com/etcd-io/etcd/releases)
etcdctl –endpoints=http://<MASTER-IP>:2379 get / –prefix –keys-only
#Kubelet
curl http://<external-IP>:10255/pods

3. Check images version
kubectl get deployments -A -o yaml | grep "image:"

4. Check services
kubectl get services -A -o wide

5. Check deployments
kubectl get deployments -A -o yaml > deployments.yaml
checkov -f deployments.yaml

6. Check permissions
kubectl auth can-i --list (-n <namespace)

7. Look for sensitive information(in pods)

8. Inspect the Tokens
#Dump tokens from inside the pod
kubectl exec -ti <pod> -n <namespace> cat /run/secrets/kubernetes.io/serviceaccount/token
#Dump all tokens from secrets
kubectl get secrets -A -o yaml | grep " token:" | sort | uniq > alltokens.txt

9. Check out if they can read secrets from other namespaces
curl -v -H "Authorization: Bearer <jwt_token>" https://<master_ip>:<port>/api/v1/namespaces/<namespace>/secrets/

Tools
Kubebox, kube-bench, kube-goat, kube-hunter
![image](https://user-images.githubusercontent.com/92818014/137967712-b39631a7-5b5d-400f-8467-93f40cb2563a.png)

