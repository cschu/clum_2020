# de.NBI cloud user meeting 2020 - CLUM 2020
Teaching aids / [slides for the de.NBI cloud user meeting 2020]()  

## Setup of BiBiGrid

### Requirements

BiBiGrid is written in Java and needs therefor a Java Runtime Environment (>= 8) installed. Additionally a terminal and a ssh-client is needed to work with BiBiGrid

### Download 
The easiest and recommendend way to use BiBiGrid is to download the [latest prebuild binary](https://bibiserv.cebitec.uni-bielefeld.de/resources/bibigrid/bibigrid-openstack-2.2.jar).  

### Build from Source
Alternatively, you may clone the [BiBiGrid repository](https://github.com/BiBiServ/bibigrid/) and build it by yourself using maven, which requires a Java Development Kit (>= 8) and maven (>= 3.9) installed.

``` BASH
> git clone https://github.com/BiBiServ/bibigrid.git
> cd bibigrid
> mvn -P openstack clean package
```

### Credentials
BiBiGrid needs access to the Openstack API to work properly. 

#### Get API Access 

The access to the de.NBI cloud sites web-based user interface (Openstack Dashboard) is realized by a SSO mechanism using Elixir AAI. **An API password is not set by default.**

#### Download Openstack RC file 

The OpenStack RC file is a file that contains the environment variables that are necessary to run OpenStack command-line clients. The file contains project-specific environment variables and allows access to the Openstack API. After login into the Openstack Dashboad you can download the *OpenStack RC File v3*  by clicking on your account symbol on the right upper corner.  

![Pop-Up Menü oben rechts](images/popup_rc-file.png)  

After downloading you have to open up a terminal and source the downloaded file (e.g. clum2020-openrc.sh) to get the credentials into your environment.  

```BASH
> source FILE.sh
```
  
_Note: You have to source the RC file in every new terminal, if you want to access OpenStack API._  

_Note: Application credentials unfortuneatly not an option, because Openstack4J - the library used by BiBiGrid to talk with Openstack - does not support Application credentials._

### Configuration

The prefilled configuration template below works on de.NBI cloud site Bielefeld with m at time of writing (10/5/2020). You have to adjust most values when trying this on other de.NBI cloud site.

#### template

```JSON
#use openstack
mode: openstack

#Access
sshUser: ubuntu
region: Bielefeld                            
availabilityZone: default                         

#Network
subnet: XXXXXX                                     # REPLACE

# Master instance
masterInstance:
  type: de.NBI default + ephemeral
  image: e4ff922e-7681-411c-aa9b-6784390a904e
  
# Worker instances
workerInstances:
  - type: de.NBI small + ephemeral 
    image: e4ff922e-7681-411c-aa9b-6784390a904e
    count: 3 
useMasterWithPublicIp: yes
useMasterAsCompute: no

#services
nfs: yes
zabbix: yes
slurm: yes

ideConf:
  ide: true
  
zabbixConf:
    admin_password: XXXXX                          # REPLACE
``` 

1. [Download](configuration.yml) the prefilled configuration template
2. Open it with an editor by your choice and replace the **XXXXXXX** values for `subnet` and `zabbixConf.admin_password`

#### access
BibiGrid creates a new SSH key-pair (stored at `~/.bibigrid/keys`) for each cluster started. This cluster specific keys are used to connect to master instance. Additionally it is possible to add additional SSH-keys for communication with the remote compute system (not covered by our template above, see BiBigrid documentation for a precise description).\

#### ssh-user
The ssh-user depends on the cloud image your cluster is based on. Since we run on top of Ubuntu 18.04 the ssh-user is ubuntu.

#### region
The region is can be determined easily by running the openstack cli.

```bash
$ openstack region list
```

#### availability zone
The availability zone where your instances are created.

```bash 
$ openstack availability zone list
```

#### network
If you have the permissions to create networks, BiBiGrid offers the possibility to create a new private network connected to an existing router. For our tutorial we work on an existing subnet. Determine the subnet name or id using the cli.

```BASH
$ openstack subnet list
```

#### instances
We want to use a default Ubuntu 18.04 image for our tutorial. Determine the id of it using the cmdline client ...

```BASH
$ openstack image list
```

... and add it to the master/worker configuration. 


#### services
We use a typical cluster configuration for our workshop setup. That means we have to enable a shared fs (nfs), a grid batch scheduler (slurm), a monitoring framework (zabbix) and a web-ide (theia). 


### Creating a bibigrid alias
To keep the cluster setup process simple you can set an alias for the BiBiGrid JAR file installed before.  
The Unix command should look like the following (depending on JAR filename):

```BASH
> alias bibigrid="java -jar /path/to/bibigrid-*.jar"
```

### Verfify the configuration
You can simply check your configuration using :

```BASH
> bibigrid -o configuration.yml -ch
```

### BiBiGrid commands - Start your first Cluster
For information about the command set, you may now check the help command:  

```BASH
> bibigrid -o configuration.yml --help
```

Now we can create the first cluster with our previously generated configuration:  

```BASH
> bibigrid -o configuration.yml -c -v 
```

If no problem occurs, our cluster should be ready to work with in about 15 minutes ... time for a coffee break  

![coffee break](images/coffee_break.jpg)

Since it is possible to start more than one cluster at once, it is possible to list all running clusters (within the same Openstack project):  

```BASH
> bibigrid -o configuration.yml --list
```

The command returns an informative list about all your running clusters.

### Good to know

#### SLURM
SLUM is an open source and scalable cluster management and job scheduling system for large and small Linux clusters. As a cluster workload manager, Slurm has three key functions. First, it allocates access to resources on the worker nodes to users for some duration of time so they can perform work. Second, it provides a framework for starting, executing, and monitoring work (normally a parallel job) on the set of allocated workers. Finally, it arbitrates contention for resources by managing a queue of pending work. (See [documentation](https://slurm.schedmd.com/) for a detailed documentation).

##### User commands (partial list)

- `scontrol`: View and/or modify Slurm state
- `sinfo   `: Reports state of Slurm partitions and nodes
- `squeue  `: Reports the state of jobs or job steps
- `scancel `: Cancel a pending or running job or job step
- `srun    `:  Submit a job for execution
- ...

#### Directory structure
BiBigrid span a shared filesystem (NFS) between all cluster node. The master acts a NFS server and all clients connects to it.

- `/vol/spool` -> shared filesystem between all nodes.
- `/vol/scratch` -> local diskspace (ephemeral disk, if provided)

## Login into the cluster

After a successfull setup ...

```BASH
SUCCESS: Cluster has been configured. 
Ok : 
 You might want to set the following environment variable:

export BIBIGRID_MASTER=129.70.51.XXX

You can then log on the master node with:

ssh -i /Users/jkrueger/.bibigrid/keys/bibigridther0ysuts6vo37 ubuntu@$BIBIGRID_MASTER

The cluster id of your started cluster is: ther0ysuts6vo37

You can easily terminate the cluster at any time with:
bibigrid -t ther0ysuts6vo37 

```

... you can login into the master node. Run `sinfo` to check if there are 3 worker available.

## Login into the cluster (more comfortable)

BiBiGrid offers a more comfortable way to work with your cloud instances using the WEB-IDE [theia](https://theia.org).
Let's see how this works together with BiBiGrid.

![Theia WebIDE Terminal](images/theia_ide_terminal.png)

If the ide option is enabled in the configuration, theia will be run as systemd service on localhost. For security reasons, theia is not binding to a standard network device. A valid certificate and some kind of authentication is needed to create a safe connection, which is not that easy in a dynamic cloud environment.

However, BiBiGrid has the possibility to open a ssh tunnel from the local machine to BiBiGrids master instance and open up a browser window running theia web ide.

```bash
bibigrid -o configuration.yml --ide <cluster id>
```



## Hello World, Hello BiBiGrid!

To see how the cluster with slurm works in action, we start with a typical example : *Hello World !*

- If not already done, connect to your cluster (via terminal or Web-IDE) 

- Create a new shell script `helloworld.sh` in the spool directory (`/vol/spool`):

```
#!/bin/bash
echo Hello from $(hostname) !
sleep 10
```

- Open a terminal and change into the spool directory. 
`cd /vol/spool`


- Make our helloworld script executable:
`chmod u+x helloworld.sh`

- Submit this script as array job 50 times : `sbatch --array=1-50 --job-name=helloworld hello-world.sh`
- See the status of our cluster: `squeue`
- See the output: `cat slurm-*.out`

## Manual Cluster Scaling (NEW)
In some cases, you may want to scale down your cluster, if you don't need as much worker instances or scale up, if you want to append some.
We scale down one worker instance of our first worker batch previously configured.

```BASH
> bibigrid -o configuration.yml -sd <bibigrid-id> 1 1
```

Scaling down is quite fast, only the master node have to be reconfigured. Check the number of working nodes running `sinfo` on the master node. Since we need three workers for the 2nd part of this workshop we now scale up one worker instance ...

```BASH
> bibigrid -o configuration.yml -su <bibigrid-id> 1 1
```

... and check again the number of worker nodes using `info`. Scaling up takes some time (a few minutes) since newly added worker now must be configured from scratch.

*Note: If you have a running cluster, that is barely working to full capacity, it is recommended to scale it down, since there are currently no checks for qeued jobs.*

To scale up 

## Monitoring your cluster setup
To get an overview about how your cluster is working, you can use *Zabbix* for monitoring.  
Therefore it is necessary to use a port-forwarding in order to access the zabbix server on your local browser.

Login to the cluster: 

```bash
ssh -L <local-port>:localhost:80 user@ip-address
```

As the `<local-port>` you have to choose a free port on local system (e.g. 8080).  
The `ìp-address` is the public address of your master instance, that you already got back after cluster finish in the line `export BIBIGRID_MASTER=<ip-address>`.   Alternatively, you can use the `list` command from above to get an overview and copy the respective address in the row `public-ip`.

After you have successfully logged in into your master instance, type `http://localhost:<local-port>/zabbix` into your browser address bar. Accordingly, it is the same `<local-port>` that you have chosen before.   
The public-ip of your cluster should be visible with the list command `bibigrid -l` and is also displayed after setup.

![Zabbix Login](images/zabbix_login.png)

You can login with the 'admin' user and the previously set admin password.

For a detailed documentation please visit the [Getting Started Readme](https://github.com/BiBiServ/bibigrid/blob/master/docs/README.md).


##  Using Ansible and run a bigger example

Ansible is used by BiBiGrid to configure all started instances. Therefore it can also be used to modify an existing cluster.

-> [Ansible in a nutshell](https://gitlab.ub.uni-bielefeld.de/denbi/ansible-course)

## Terminate a cluster

To terminate a running cluster you can simply use:

```bash
bibigrid -o configuration.yml -t <clusterid>
```

Optionally, it is possible to terminate more than one cluster appending the other ids as follows:

```bash
bibigrid -o configuration.yml -t <clusterid1> <clusterid2> <clusterid3> ...
```

Another option is to terminate all your clusters using your username:

```bash
bibigrid -o configuration.yml -t <user>
```
