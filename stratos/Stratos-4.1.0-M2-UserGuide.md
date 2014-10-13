Stratos 4.1.0 - M2 - Developer Preview - Getting Started Guide
=========

Table of Content
----------

- [Main Features](#main-features)
- [Pre-requisite](#pre-requisite)
- [Testing M2](#testing-m2)
- [Jira List](#jira-list)
- [Troubleshooting Guide](#troubleshooting-guide)


Main Features
-------------

- [Docker](https://www.docker.com/) support using [Google Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes) and [CoreOS](https://coreos.com/)
- [MQTT](http://mqtt.org/) support (removal of JNDI)
- Python Cartridge Agent 

Pre-requisite
-------------

- Setting up the Kubernetes-CoreOS cluster in your local machine
    * Install [Vagrant 1.6.5](https://www.vagrantup.com/) and [Oracle VM VirtualBox Manager 4.3.14](https://www.virtualbox.org/) in your local machine.
    * Get a GIT clone of [Vagrant Kubernetes Setup](https://github.com/nirmal070125/vagrant-kubernetes-setup) onto your machine.
    * Navigate to the cloned repository directory (**SETUP_HOME**).
    * Run ``` up.sh ``` script file - ``` {SETUP_HOME}$ ./up.sh ``` (You might have to use ```sudo``` in some cases.)
    * Above command will start-up 4 VMs, namely discovery, master, minion-1 and minion-2.
    * SSH to master node ; ``` {SETUP_HOME}$ vagrant ssh master ```
    * Pull Stratos PHP Docker Image from DockerHub into master node or into the local machine.
    ``` sh 
    docker pull apachestratos/php:4.1.0-m2 
    ```
    * Import downloaded Stratos PHP Docker image as a tarball.
    ```sh
    docker save -o stratos-php-latest.tar  apachestratos/php:4.1.0-m2 
    ```     
    * SCP the Stratos PHP Docker Image tarball to minion-1 and minion-2. You can find the private key file which you can use to SCP, in the **{SETUP_HOME}/ssh.config** file, against **IdentityFile** attribute. 
    ``` sh
    scp -i ~/.vagrant.d/insecure_private_key stratos-php-latest.tar core@172.17.8.101:.
    ```
    * SSH to minion-1 (``` {SETUP_HOME}$ vagrant ssh minion-1 ```) and minion-2 (``` {SETUP_HOME}$ vagrant ssh minion-2```)
    * Load Stratos PHP Docker Image from tarball to minion-1 and minion-2
    ```sh
    docker load -i stratos-php-latest.tar
    ```   
    * Verify that the image is properly loaded by issuing ```docker images``` in each node;
    ```sh
    core@master ~ $ docker images
    REPOSITORY                   TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    apachestratos/php:4.1.0-m2   latest              0fff8e5ac572        3 hours ago         452.1 MB
    ```

- Download and extract [Apache ActiveMQ 5.10.0 or later](http://activemq.apache.org/) and start ActiveMQ - ``` {ACTIVEMQ_HOME}$ ./bin/activemq start ```
  Please make sure mqtt transport connector is enabled in the ActiveMQ configuration file; **{ACTIVEMQ_HOME}/conf/activemq.xml**.

- Build Stratos master code, copy (from {**STRATOS_SOURCE}/products/stratos/modules/distribution/target/**) and extract the binary **apache-stratos-4.1.0-SNAPSHOT.zip** to a preferred directory (**STRATOS_HOME**). 

- Change the **MgtHostName** and **HostName** elements' values in **{STRATOS_HOME}/repository/conf/carbon.xml** such that they point to the private IP address of your local machine.

- Change the **dataBridgeConfiguration.thriftDataReceiver.hostName** element's value in **{STRATOS_HOME}/repository/conf/data-bridge/data-bridge-config.xml** to the private IP address of your machine.

- Change the property named **"java.naming.provider.url"** value to **tcp://{MB_IP}:61616** in **{STRATOS_HOME}/repository/deployment/server/outputeventadaptors/JMSOutputAdaptor.xml** file.

- Change the **expiryTimeout** element's value in **{STRATOS_HOME}/repository/conf/autoscaler.xml** to **30000** (it is the maximum time a member can be in the pending state)

- Start Stratos using ``` {STRATOS_HOME}$ ./bin/stratos.sh start ``` command.


Testing M2
----------

##1. Register Kubernetes-CoreOS Host Cluster in Stratos

### Curl Command
``` sh 
curl -X POST -H "Content-Type: application/json" -d @"kub-register.json" -k  -u admin:admin "https://localhost:9443/stratos/admin/kubernetes/deploy/group"
```

### CLI Command
```bash
deploy-kubernetes-group -p kubernetes-group.json
```

###kub-register.json
```javascript
{
      "groupId": "KubGrp1",
      "description": "Kubernetes CoreOS cluster on EC2 ",
      "kubernetesMaster": {
                  "hostId" : "KubHostMaster1",
                  "hostname" : "master.dev.kubernetes.example.org",
                  "hostIpAddress" : "172.17.8.100",
                  "property" : [
                      {
                         "name": "prop1",
                         "value": "val1"
                      },
                      {
                         "name": "prop2",
                         "value": "val2"
                      }
                  ]
        },

        "portRange" : {
           "upper": "5000",
           "lower": "4500"
        },

        "kubernetesHosts": [
              {
                     "hostId" : "KubHostSlave1",
                     "hostname" : "slave1.dev.kubernetes.example.org",
                     "hostIpAddress" : "172.17.8.101",
                     "property" : [
                         {
                             "name": "prop1",
                             "value": "val1"
                         },
                         {
                             "name": "prop2",
                             "value": "val2"
                         }
                     ]
                },
                {
                     "hostId" : "KubHostSlave2",
                     "hostname" : "slave2.dev.kubernetes.example.org",
                     "hostIpAddress" : "172.17.8.102",
                     "property" : [
                         {
                             "name": "prop1",
                             "value": "val1"
                         },
                         {
                             "name": "prop2",
                             "value": "val2"
                         }
                     ]
                }
    ],

    "property": [
           {
                  "name": "prop1",
                  "value": "val1"
           },
           {
                  "name": "prop2",
                  "value": "val2"
           }
    ]
}
```

### Verify Kubernetes-CoreOS Host Cluster Registration

###Curl Command
```sh
 curl  -k  -u admin:admin "https://localhost:9443/stratos/admin/kubernetes/group/KubGrp1"
```

####Response:
```javascript
{"kubernetesGroup":{"description":"Kubernetes CoreOS cluster on EC2 ","groupId":"KubGrp1","kubernetesHosts":[{"hostId":"KubHostSlave1","hostIpAddress":"172.17.8.101","hostname":"slave1.dev.kubernetes.example.org","property":[{"name":"prop1","value":"val1"},{"name":"prop2","value":"val2"}]},{"hostId":"KubHostSlave2","hostIpAddress":"172.17.8.102","hostname":"slave2.dev.kubernetes.example.org","property":[{"name":"prop1","value":"val1"},{"name":"prop2","value":"val2"}]}],"kubernetesMaster":{"hostId":"KubHostMaster1","hostIpAddress":"172.17.8.100","hostname":"master.dev.kubernetes.example.org","property":[{"name":"prop1","value":"val1"},{"name":"prop2","value":"val2"}]},"portRange":{"lower":4000,"upper":5000},"property":[{"name":"prop1","value":"val1"},{"name":"prop2","value":"val2"}]}}
```

### CLI Command
```bash
list-kubernetes-groups
```

#### Response
```
Kubernetes groups found:
+----------+-----------------------------------+
| Group ID | Description                       |
+----------+-----------------------------------+
| KubGrp1  | Kubernetes CoreOS cluster on EC2  |
+----------+-----------------------------------+
```

### CLI Command
```bash
list-kubernetes-hosts KubGrp1
```

#### Response
``` 
Kubernetes hosts found:
+---------------+-----------------------------------+--------------+
| Host ID       | Hostname                          | IP Address   |
+---------------+-----------------------------------+--------------+
| KubHostSlave1 | slave1.dev.kubernetes.example.org | 172.17.8.101 |
+---------------+-----------------------------------+--------------+
| KubHostSlave2 | slave2.dev.kubernetes.example.org | 172.17.8.102 |
+---------------+-----------------------------------+--------------+
```

##2. Deploy a Docker Cartridge

###Curl Command
``` sh 
curl -X POST -H "Content-Type: application/json" -d @'php-docker-cartridge.json' -k -v -u admin:admin "https://localhost:9443/stratos/admin/cartridge/definition"
```

### CLI Command
```bash
deploy-cartridge -p php-docker-cartridge.json
```

###php-docker-cartridge.json
```javascript
{
      "type": "php",
      "provider": "apache",
      "host": "apachestratos.org",
      "displayName": "PHP",
      "description": "PHP Cartridge",
      "version": "5.0",
      "multiTenant": "false",
      "deployerType": "kubernetes",
      "portMapping": [
         {
            "protocol": "http",
            "port": "80",
            "proxyPort": "8280"
         }
       ],
       "container": [
        {
          "imageName": "apachestratos/php:4.1.0-m2",
          "property": [
            {
             "name": "prop-name",
             "value": "prop-value"
            }
          ]
        }
      ]
 }
```

##3. Deploy the autoscale policy

###Curl Command
```bash
curl -X POST -H "Content-Type: application/json" -d @'autoscale-policy.json' -k -v -u admin:admin “https://localhost:9443/stratos/admin/policy/autoscale”
```

### CLI Command
```bash
deploy-autoscaling-policy -p autoscale-policy.json
```

###autoscale-policy.json
```javascript
{
    "id": "economy",
    "loadThresholds": {
      "requestsInFlight": {
         "upperLimit": 80,
         "lowerLimit": 5
      },
      "memoryConsumption": {
         "upperLimit": 80,
         "lowerLimit": 15
      },
      "loadAverage": {
         "upperLimit": 180,
         "lowerLimit": 20
      }
    }
}
```

##4. Subscribe to a Docker Cartridge

### Curl Command
``` sh 
curl -X POST -H "Content-Type: application/json" -d @php-subscription.json -k -v -u admin:admin "https://localhost:9443/stratos/admin/cartridge/subscribe"
```

### CLI Command
```bash
subscribe-cartridge --autoscaling-policy economy -p php-cartridge-subscription.json
```


###php-subscription.json

- Replace **payload_parameter.MB_IP** and **payload_parameter.CEP_IP** by your local machine IP in the following json. Add any additional payload parameters that are needed to get PHP cartridge running, such as the repository information etc.

```javascript
{
    "cartridgeType": "php",
    "alias": "myphp",
    "commitsEnabled": "false",
    "autoscalePolicy": "economy",
    "repoURL": "https://github.com/sajhak/myphprepo",
    "repoUsername": "sajhak@gmail.com",
    "repoPassword": "sasa200",
    "property": [
            {
             "name": "KUBERNETES_CLUSTER_ID",
             "value": "KubGrp1"
            },
	        {
             "name": "KUBERNETES_REPLICAS_MIN",
             "value": "3"
            },     
            {
             "name": "KUBERNETES_REPLICAS_MAX",
             "value": "20"
            }, 
            {
             "name": "payload_parameter.MB_IP",
             "value": "10.100.5.84"
            },       
            {
             "name": "payload_parameter.MB_PORT",
             "value": "1883"
            },       
            {
             "name": "payload_parameter.CEP_IP",
             "value": "10.100.5.84"
            },       
            {
             "name": "payload_parameter.CEP_PORT",
             "value": "7611"
            },       
            {
             "name": "payload_parameter.LOG_LEVEL",
             "value": "DEBUG"
            },
            {
             "name": "payload_parameter.APP_PATH",
             "value": "/var/www"
            }
          ]    
}

```

##5. Unsubscribe from a Cartridge

###Curl Command
```sh
curl -X POST -H "Content-Type: application/json" -d 'myphp' -k -v -u admin:admin "https://localhost:9443/stratos/admin/cartridge/unsubscribe"
```

### CLI Command
```bash
unsubscribe-cartridge myphp
```

##6. Accessing PHP service

Currently accessing via Load Balancer is not supported. You could access the service via **http://{HOST_IP}:{SERVICE_PORT}**.

In order to find the **SERVICE_PORT** issue following command in the Kubernetes Master Node.
```sh
kubecfg list services
```

Then access from one of the following URLs:
- http://172.17.8.100:{SERVICE_PORT}
- http://172.17.8.101:{SERVICE_PORT}
- http://172.17.8.102:{SERVICE_PORT}


##7. Testing Autoscaling

Currently autoscaling works based on CPU and Memory usage. You can stress docker containers using stress tool.

- ssh to the coreos node which is having containers (see trouble shoot guide at the end)

- Install stress tool
```sh
apt-get install stress
```
- stress the container
```sh
stress -c 4
```
- observe the stratos log, you will get drools logs regarding scaling

- check the number of pods in master node
```sh
kubecfg list /pods
```


Jira List
----------

    
**Sub-task**

<ul>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-734'>STRATOS-734</a>] -         General Improvements to Stratos in docker image
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-764'>STRATOS-764</a>] -         use a maven docker plugin to build stratos docker images
</li>
</ul>
            
**Bug**

<ul>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-377'>STRATOS-377</a>] -         Getting &quot;Tenant not found&quot; warning when subscribing cartridges as super tenant   
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-589'>STRATOS-589</a>] -         [vCloud] Getting error when unsubscribing, but instance terminated successfully
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-590'>STRATOS-590</a>] -         DHCP based IP assignment not working on vCloud environment
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-641'>STRATOS-641</a>] -         LoadBalancer doesn&#39;t keep super-tenant subscriptions for a multi-tenant service
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-705'>STRATOS-705</a>] -         create docker files for stratos products and activemq/mysql
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-723'>STRATOS-723</a>] -         Stratos stopped working with java.net.SocketException
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-771'>STRATOS-771</a>] -         [Wiki] Auto commit option in UI not documented
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-775'>STRATOS-775</a>] -         Error when trying to login as a tenant from Carbon UI
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-802'>STRATOS-802</a>] -         Partition deployment fails in EC2
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-814'>STRATOS-814</a>] -         Tenant admin permissions are persisted in UM_PERMISSION incorrectly.
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-818'>STRATOS-818</a>] -         Error log getting printed during the server start-up
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-882'>STRATOS-882</a>] -         [CLI] Invalid Error Handling at Login
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-883'>STRATOS-883</a>] -         CLI does not show error messages when unknown server errors occur
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-886'>STRATOS-886</a>] -         List Kubernetets Hosts method does to work in REST API
</li>
</ul>
                
**Improvement**

<ul>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-603'>STRATOS-603</a>] -         Validate Git URL
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-695'>STRATOS-695</a>] -         [Wiki] - Improve Cartridge Documentation
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-813'>STRATOS-813</a>] -         Fix critical issues reported by Sonar
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-857'>STRATOS-857</a>] -         Clean/Refactor Stratos REST API to have Restful Operations 
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-873'>STRATOS-873</a>] -         [Sonar Findings] [Critical] Array is Stored Directly 
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-875'>STRATOS-875</a>] -         Abstracting out the Application and Group in the Topology
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-884'>STRATOS-884</a>] -         CLI has duplicated command implementations in RestCommandLineService class
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-885'>STRATOS-885</a>] -         CLI command classes are not using a proper naming convention
</li>
</ul>
    
**New Feature**

<ul>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-761'>STRATOS-761</a>] -         Tenant isolation for policies and definitions - in Autoscalar
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-785'>STRATOS-785</a>] -         Autoscaling Containers in Stratos
</li>
</ul>
                                
**Task**

<ul>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-856'>STRATOS-856</a>] -         Need a wiki page for Performance Tuning
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-867'>STRATOS-867</a>] -         Tenant isolation for Kubernetes definitions - in Autoscalar
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-868'>STRATOS-868</a>] -         Add mock REST endpoint methods for Kubernetes 
</li>
<li>[<a href='https://issues.apache.org/jira/browse/STRATOS-869'>STRATOS-869</a>] -         Add exposed Kubernetes related methods in CLI
</li>
</ul>



Troubleshooting Guide
---------------------


**1. How to ssh to master node?**

- Navigate to the cloned repository directory (**SETUP_HOME**)
```sh
vagrant ssh master
```

**2. How to ssh to minion-1 node?**

- Navigate to the cloned repository directory (**SETUP_HOME**)
```sh
vagrant ssh minion-1
```

**3. How to ssh to minion-2 node?**

- Navigate to the cloned repository directory (**SETUP_HOME**)
```sh
vagrant ssh minion-2
```

**4. How to list all the machines in CoreOS cluster?**

- ssh to master node
```sh
fleetctl list-machines
```

- output
```sh
core@master ~ $ fleetctl list-machines
MACHINE		IP		METADATA
07215782...	172.17.8.100	-
4b56425a...	172.17.8.102	-
bf39a4c4...	172.17.8.101	-
```


**5. How to list the available replication controllers?**

- ssh to master node
```sh
kubecfg list /replicationControllers
```
- output
```sh
core@master ~ $ kubecfg list /replicationControllers
ID                  Image(s)                           Selector         Replicas
test2.php.domain    54.254.64.141:5000/stratos-php     name=php         2
```


**6. How to list the available pods?**

- ssh to master node
```sh
kubecfg list /pods
```
- output
```sh
core@master ~ $ kubecfg list /pods
ID                                     Image(s)     Host                Labels                              Status
115bbe15-49ff-11e4-91b7-08002794b041   stratos-php  172.17.8.100/      name=php,replicationController=php   Waiting
115cc7e9-49ff-11e4-91b7-08002794b041   stratos-php  172.17.8.101/      name=php,replicationController=php   Waiting
```

**7. How to tail the kubernetes log?**

- ssh to master node
```sh
journalctl -f
```

**8. How to restart the kubernetes scheduler?**

- ssh to master node
```sh
systemctl restart scheduler
```

**9. How to list all the docker containers in a node?**

- ssh to the node
```sh
docker ps
```
- output
```sh
core@master ~ $ docker ps
CONTAINER ID        IMAGE                        COMMAND                CREATED             STATUS   PORTS
37e3303eb337        stratos-php:latest           "/bin/sh -c '/usr/lo   2 minutes ago       Up                
a3f787d0a7ae        kubernetes/pause:latest      "/pause"               2 minutes ago       Up       0.0.0.0:80->80/tcp
```

**10. How to get the IPAddress of a docker container?**

- ssh to the node
```sh
docker inspect CONTAINER-ID | grep IPAddress
```
- output
```sh
core@master ~ $ docker inspect a3f787d0a7ae | grep IPAddress
IPAddress": "10.100.56.3",
```

**10. How to ssh to a docker container?**

- ssh to the node
```sh
ssh root@CONTAINER-IPAddress
```
- enter ```g``` for password

**11. How to kill a docker container?**

- ssh to the node
```sh
docker kill CONTAINER-ID
```
- output
```sh
core@minion-1 ~ $ docker kill 6f5ba525f9ab
6f5ba525f9ab
```
- another conatiner will be created within few seconds

**12. How to get info of a specific replicationController as a json?**

- ssh to the master node
```sh
 kubecfg -json get /replicationControllers/REPLICATION-CONTROLLER-ID
```
- output
```"
core@master ~ $  kubecfg -json get /replicationControllers/test2.php.domain
{"kind":"ReplicationController","id":"test2.php.domain","creationTimestamp":"2014-10-02T07:44:58Z","resourceVersion":6701,"apiVersion":"v1beta1","desiredState":{"replicas":2,"replicaSelector":{"name":"test2.php.domain"},"podTemplate":{"desiredState":{"manifest":{"version":"v1beta1","id":"","volumes":null,"containers":[{"name":"test2-apachestratos-org","image":"54.254.64.141:5000/stratos-php","ports":[{"name":"tcp80","hostPort":80,"containerPort":80,"protocol":"tcp"}],"env":[{"name":"SERVICE_NAME","key":"SERVICE_NAME","value":"php"},{"name":"HOST_NAME","key":"HOST_NAME","value":"test2.apachestratos.org"},{"name":"MULTITENANT","key":"MULTITENANT","value":"false"},{"name":"TENANT_ID","key":"TENANT_ID","value":"-1234"},{"name":"TENANT_RANGE","key":"TENANT_RANGE","value":"-1234"},{"name":"CARTRIDGE_ALIAS","key":"CARTRIDGE_ALIAS","value":"test2"},{"name":"CLUSTER_ID","key":"CLUSTER_ID","value":"test2.php.domain"},{"name":"CARTRIDGE_KEY","key":"CARTRIDGE_KEY","value":"uLBSXhS3Kzos5xVe"},{"name":"REPO_URL","key":"REPO_URL","value":"null"},{"name":"PORTS","key":"PORTS","value":"80"},{"name":"PROVIDER","key":"PROVIDER","value":"apache"},{"name":"PUPPET_IP","key":"PUPPET_IP","value":"127.0.0.1"},{"name":"PUPPET_HOSTNAME","key":"PUPPET_HOSTNAME","value":"puppet.raj.org"},{"name":"PUPPET_DNS_AVAILABLE","key":"PUPPET_DNS_AVAILABLE","value":"null"},{"name":"PUPPET_ENV","key":"PUPPET_ENV","value":"stratos"},{"name":"DEPLOYMENT","key":"DEPLOYMENT","value":"default"},{"name":"CEP_PORT","key":"CEP_PORT","value":"7611"},{"name":"COMMIT_ENABLED","key":"COMMIT_ENABLED","value":"false"},{"name":"MB_PORT","key":"MB_PORT","value":"1883"},{"name":"MB_IP","key":"MB_IP","value":"172.17.42.1"},{"name":"CEP_IP","key":"CEP_IP","value":"172.17.42.1"},{"name":"MEMBER_ID","key":"MEMBER_ID","value":"test2.php.domaindb8e4a24-17e8-40a2-ad85-e0d0ca127887"},{"name":"LB_CLUSTER_ID","key":"LB_CLUSTER_ID"},{"name":"NETWORK_PARTITION_ID","key":"NETWORK_PARTITION_ID"},{"name":"KUBERNETES_CLUSTER_ID","key":"KUBERNETES_CLUSTER_ID","value":"KubGrp1"},{"name":"KUBERNETES_MASTER_IP","key":"KUBERNETES_MASTER_IP","value":"127.0.0.1"},{"name":"KUBERNETES_PORT_RANGE","key":"KUBERNETES_PORT_RANGE","value":"4000-5000"}]}],"restartPolicy":{"always":{}}}},"labels":{"name":"test2.php.domain"}}},"currentState":{"replicas":2,"podTemplate":{"desiredState":{"manifest":{"version":"","id":"","volumes":null,"containers":null,"restartPolicy":{}}}}},"labels":{"name":"test2.php.domain"}}
```

**12. How to get update a specific replicationController?**

- ssh to the master node
- get the replication controller info as a json as above and save it to a file
```sh
 kubecfg -json get /replicationControllers/REPLICATION-CONTROLLER-ID >> rep-controller.json
```
- edit the rep-controller.json (for example, set replicas to 0)
- update the replicationController
```sh
 kubecfg -c rep-controller.json update /replicationControllers/REPLICATION-CONTROLLER-ID
```
- output
```sh
core@master ~ $ kubecfg -c raj.json update /replicationControllers/test2.php.domain
ID                  Image(s)                         Selector                Replicas
test2.php.domain    54.254.64.141:5000/stratos-php   name=test2.php.domain   0
```
- it will delete all the pods imediately


**13. How to delete a pod?**

- ssh to the master node
- get the pod ID using ```kubecfg list /pods```
```sh
kubecfg delete pods/POD-ID
```
- output
```sh
core@master ~ $ kubecfg delete pods/3ed5c5a6-4a0a-11e4-91b7-08002794b041
I1002 08:01:25.351501 02112 request.go:292] Waiting for completion of /operations/67
Status
success
```

**14. How to delete a replicationController?**

- ssh to the master node
- get the replicationController ID using ```kubecfg list /replicationControllers```
```sh
kubecfg delete replicationControllers/REPLICATION-CONTROLLER-ID
```
- output
```sh
core@master ~ $ kubecfg delete /replicationControllers/test2.php.domain
I1002 08:32:47.187198 02150 request.go:292] Waiting for completion of /operations/86
Status
success
```
- if you unsubscribe to the cartridge, replicationControllers, pods and containers for that service cluster will be wiped out
