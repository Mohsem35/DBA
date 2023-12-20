
_manager_node: 172.16.6.5(114)_
_worker_node1: 172.16.6.57(174)_
_worker_node2: 172.16.6.18(243)_

_Harbor node: 172.16.7.246_

### Docker Swarm


#### Initialize swarm manager and slave nodes 

**Step 1:** Install Docker on every nodes. 

For Docker installation, follow the official document: [Install Docker Engine on Debian](https://docs.docker.com/engine/install/ubuntu/)
> _Note_: use apt command instead of apt-get

Change hostname as manager
```shell
sudo hostnamectl set-hostname swarm-manager
```

> _Note:_ Do the same for worker nodes. change the hostmame for worker nodes

Add `sudo` permission to user
```shell
sudo usermod -a -G sudo ubuntu
sudo groupadd docker
sudo gpasswd -a $USER docker
sudo service docker restart
```

**Step 2:** Initialize the Swarm Manager at manager_node

```shell
docker swarm init --advertise-addr <IP_address_of_the_manager_node>


# output
Swarm initialized: current node (fvaeww8dtfymtyzn3vephz25h) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0oeyle8yr4sf71o3a34tk30fdruqlpkeiui3llrkbakwvzyfuu-4zatclqcn9h8rw3mxdp1xlkcl 172.16.6.5:2377
```

**Step 3:** Add worker nodes using token

To add worker_nodes to this manager_node, run **`docker swarm join-token manager`** and follow the instructions. Run the following command at worker nodes

```shell 
docker swarm join --token SWMTKN-1-0oeyle8yr4sf71o3a34tk30fdruqlpkeiui3llrkbakwvzyfuu-4zatclqcn9h8rw3mxdp1xlkcl 172.16.6.5:2377
```
- Check node list from manager_node

```shell
ubuntu@swarm-manager:~$ sudo docker node ls

# output
ID                            HOSTNAME        STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
fvaeww8dtfymtyzn3vephz25h *   swarm-manager   Ready     Active         Leader           24.0.7
um60rpxx204ng5i4iylw441ta     swarm-worker1   Ready     Active                          24.0.7
89i5s9voh39egvrk5uxkgq8ng     swarm-worker2   Ready     Active                          24.0.7
```
**Step 4:** Add registy to manager_node and worker_nodes

Edit the `daemon.json` file and add the following line in both manager and worker nodes

```shell
vi /etc/docker/daemon.json
```

```
{
"insecure-registries" : ["registry.cellosco.pe", "0.0.0.0"]
}
```
Restart the docker service and try to login
```shell
service docker restart
docker login registry.cellosco.pe
```


### Harbor

Harbor node IP: 172.16.7.246

[MRA Harbor](http://registry.cellosco.pe/account/sign-in?redirect_url=%2Fharbor%2Fprojects)

**What is Harbor?***

Harbor is an open source **registry(like docker hub)** that secures **artifacts** with policies and role-based access control, ensures images are scanned and free from vulnerabilities, and signs images as trusted. Harbor, a CNCF Graduated project, delivers compliance, performance, and interoperability to help you consistently and securely manage artifacts across cloud native compute platforms like Kubernetes and Docker. 

![Screenshot from 2023-12-20 19-22-00](https://github.com/Mohsem35/DBA/assets/58659448/e80b1186-bf93-4578-b190-e9fcb3825b43)



#### Getting started with Harbor


1. [Download and install Harbor](https://goharbor.io/docs/2.9.0/install-config/download-installer/)

> _Note_: We will go for offline installation

2. Untar the tarball of the offline download package

```shell
tar xvzf harbor-offline-installer*.tgz
```
3. Now initialize the `harbor.yml` file 
```shell
cd /harbor
```
There will be a file named `harbor.yml.tmpl` inside the harbor directory. 

- Copy that file and give name to that new file is **`harbor.yml`**

```shell
sudo su
cp harbor.yml.tmpl harbor.yml
vim harbor.yml
```
- Change the hostname to **`registry.cellosco.pe`** and comment all the lines of the **`https related config`** module
```yaml
# Configuration file of Harbo

# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname: registry.cellosco.pe

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 80

# https related config
# https:
  # https port for harbor, default is 443
  # port: 443
  # The path of cert and key files for nginx
  # certificate: /your/certificate/path
  # private_key: /your/private/key/path
```
> _Note:_ We have to set password too for **`harbor_admin_password`** and **`harbor_database_password`** 

- Now, run the **`install.sh`** bash file
```shell
bash install.sh
```

#### Docker Swarm Stack File

In Docker Swarm, a "stack file" typically refers to a **Docker Compose file** that is used to define and manage **a set of services** as a single application stack within a **Docker Swarm cluster** 

```yml
version: '3'
services:

  config-server:
    image: 192.168.50.26/mra/cloud-config@sha256:115bf42b4232d23b671042195308e5a6f1c40f16e79cc2c6e262787421c218a2
    ports:
      - 8888:8888
    deploy:
      placement:
        constraints:
          # - node.role != manager
          - node.hostname == ims-api-service-0
        # preferences:
        #   - spread: docker-worker2.cs.net
        #   - spread: docker-worker3.cs.net
      replicas: 1
    volumes:
      - mra_config:/var/log/mra/config-server
    networks:
      mra_network:


  gateway-server:
    image: 192.168.50.26/mra/gateway@sha256:5641b68772b93047e757e6010b51a7314071b255064a8b20ff24e9978e5554a9
    ports:
      - 8070:8070
    depends_on:
      - config-server
    deploy:
      placement:
        constraints:
          - node.hostname == dmz-0
      replicas: 1
    volumes:
      - mra_gateway:/var/log/mra/gateway-server
    networks:
      mra_network:


  web:
    image: 192.168.50.26/mra/web@sha256:631202402f320053db851bb5ce60c47e08740dc9d83c559136995283c4642ca3
    ports:
      - 80:80
    deploy:
      placement:
        constraints:
          - node.hostname == dmz-0
      replicas: 1
    volumes:
      - mra_web:/var/log/nginx/
    networks:
      mra_network:


  auth-server:
    image: 192.168.50.26/mra/auth@sha256:ce4ebade12ea153470fd91e0cc0286a5a09da5c7b9b609f4362d4d5301a53d93
    ports:
      - 8080:8080
    depends_on:
      - config-server
    deploy:
      placement:
        constraints:
          - node.hostname == dmz-0
      replicas: 1
    volumes:
      - mra_auth:/var/log/mra/mra-auth-server-0-0
    networks:
      mra_network:


  loanportfolio:
    image: 192.168.50.26/mra/loanportfolio@sha256:991e8082f467c77ee6af89b0d846d63cd62813d31e8f515b7d6cc057e566c9ea
    ports:
      - 8083:8083
    depends_on:
      - config-server
    deploy:
      placement:
        constraints:
          - node.hostname == ims-api-service-0
        # preferences:
        #   - spread: docker-worker2.cs.net
        #   - spread: docker-worker3.cs.net
      replicas: 1
    volumes:
      - mra_loanportfolio:/var/log/mra/loanportfolio-0
    networks:
      mra_network:


  common:
    image: 192.168.50.26/mra/common@sha256:4da71de03c1cfaf995fba683380fbb8610ce62548d74dad302091ba58a52bf93
    ports:
      - 8040:8040
    depends_on:
      - config-server
    deploy:
      placement:
        constraints:
          - node.hostname == ims-api-service-0
        # preferences:
        #   - spread: docker-worker3.cs.net
        #   - spread: docker-worker2.cs.net
      replicas: 1
    volumes:
      - mra_common:/var/log/mra/common
      - mra_common_files:/data/mraims/files
    networks:
      mra_network:


  mfi:
    image: 192.168.50.26/mra/mfi@sha256:784300c3cda1c50a1ba762e32d859e9325a71c0419909cc803f32f6f411a3eb5
    ports:
      - 8081:8081
    depends_on:
      - config-server
    deploy:
      placement:
        constraints:
          - node.hostname == ims-api-service-0
        # preferences:
        #   - spread: docker-worker3.cs.net
        #   - spread: docker-worker2.cs.net
      replicas: 1
    volumes:
      - mra_mfi:/var/log/mra/mfi-0
    networks:
      mra_network:


  report:
    image: 192.168.50.26/mra/report@sha256:7f6673b2dd10ea7c574dc985113b86f80f4eeb346499dbc5d314d10756fc214f
    ports:
      - 8898:8898
    depends_on:
      - config-server
    deploy:
      placement:
        constraints:
          - node.hostname == ims-api-service-0
        # preferences:
        #   - spread: docker-worker3.cs.net
        #   - spread: docker-worker2.cs.net
      replicas: 1
    volumes:
      - mra_report:/var/log/mra/report-0
    networks:
      mra_network:


volumes:
  mra_report:
  mra_mfi:
  mra_common:
  mra_loanportfolio:
  mra_auth:
  mra_web:
  mra_gateway:
  mra_config:
  mra_common_files:

networks:
    mra_network:
      driver: overlay
```
When running Docker Engine in swarm mode, you can use `docker stack deploy` to deploy a complete application stack to the swarm. The `deploy` command accepts a stack description in the form of a Compose file.

The docker stack deploy command supports any Compose file of version "3.x".

```shell
# manager_node
docker stack deploy -c stack-file.yml <given_stack_app_name>
```
- stack list
```shell
# manager_node
docker stack ls
```
- service list
```shell
# manager_node
docker service ls
```


### CICD
