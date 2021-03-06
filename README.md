# microk8s gopaddle addon

Note: in the below, the following terms are used interchangeably to
indicate the 'gopaddle addon' on microk8s:
- <i>gopaddle Lite addon</i>
- <i>gopaddle-lite addon</i>
- <i>gp-lite addon</i>

## Pre-requisites

1. OS distribution: Ubuntu 18.04; MicroK8s version: 1.24; Helm version: v3.7.1

2. gopaddle Lite add-on is supported only on a single node microk8s cluster

3. System resource requirements: 8 vCPU, 32 GB RAM, 50 GB Disk

4. 'snap' tool must already be installed.

If not installed, the following steps can be used on Ubuntu 18.04:
```
$ sudo apt-get install snapd -y
$ sudo snap install core
```

5. Set the path for snap tool to be executed as a command:
```
export PATH=$PATH:/snap/bin
```

6. microk8s must already be installed and must be running.

If not installed, use the below step to install the same:
```
$ sudo snap install microk8s --classic --channel=1.24
```

If already installed, you may want to refresh microk8s:
```
$ sudo snap refresh microk8s --channel=1.24
```

7. Check and ensure that microk8s service is running:
```
$ sudo microk8s status --wait-ready
```

you should see output like the following:
```
microk8s is running
...
```

## Steps to install gopaddle addon for microk8s

1. Add gopaddle addon repo in microk8s:
```
$ sudo microk8s addons repo add gp-lite https://github.com/gopaddle-io/microk8s-community-addons.git
```

2. Check microk8s gopaddle addon is added
```
$ sudo microk8s status
```

Among others, you should see the following listed:
```
    ...
    gopaddle-lite        # (gp-lite) Cheapest, fastest and simplest way to modernize your applications
    ...
```

## Steps to enable gopaddle addon for microk8s

1. Enable gopaddle addon in microk8s:
```
$ sudo microk8s enable gopaddle-lite
```

#### Default values:
By default, the latest gopaddle-lite version is installed, which is currently 4.2.3.

An IP address is required to access the gopaddle lite end point. The default
IP address is determined in the order mentioned below:  
- If a static IP address has been configured, this is chosen as the IP address for the access end point
- If microk8s cluster is configured with an External/Public IP address, this is chosen as the IP address for the access end point
- Else, the Internal/Private IP address configured for the microk8s cluster is used as the IP address for the access end point

Example:
```
$ sudo microk8s enable gopaddle-lite
Infer repository gp-lite for addon gopaddle-lite
Static IP input is not provided. External IP is not set for the microk8s node. Assuming Internal IP of the microk8s node for the gopaddle access endpoint.
...
Waiting for the gopaddle services to move to running state. This may take a while.
...
gopaddle lite is enabled

gopaddle lite access endpoint
http://<InternalIP|PrivateIP>:30003
```

The default 'private/internal' IP address of the microk8s node in this example was:
```
127.0.0.1 (i.e., 'localhost')
```

Note: The IP address configured for the microk8s cluster can be determined using the 'cluster-info' command of kubectl in microk8s as follows:
```
$ microk8s.kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:16443
CoreDNS is running at https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

The IP address shown against the Kubernetes control plane above is the IP address configured for the microk8s cluster.


2. Wait for ready state

Before you can use all the gopaddle services, they need to be in Ready state.
To check and wait until all the services move to Ready state, use the below
command:

```
$ sudo microk8s.kubectl wait --for=condition=ready pod -l released-by=gopaddle -n gp-lite
```

The following is a sample output when the gopaddle services are in ready state:
```
pod/rabbitmq-0 condition met
pod/influxdb-0 condition met
pod/esearch-0 condition met
pod/redis-8564f6b9fd-zqb2q condition met
pod/mongodb-0 condition met
pod/appworker-7b687d86f6-hxp8s condition met
pod/gpcore-6bc47c5c94-kq9jk condition met
pod/costmanager-564c95fcdf-x7f2t condition met
pod/clustermanager-d95cccbc-dhkl9 condition met
pod/deploymentmanager-7967f54468-qw24m condition met
pod/nodechecker-7ddfb5b556-pb9xm condition met
pod/domainmanager-7c6c6f57f7-xfn2j condition met
pod/marketplace-97bfcb68f-lnmnq condition met
pod/configmanager-5c6878bc99-8pzw7 condition met
pod/activitymanager-b7d669fb8-pcnhn condition met
pod/appscanner-677cd5799-ztrxj condition met
pod/usermanager-796bf9c8c9-f8tgg condition met
pod/cloudmanager-6c8dd7c6c5-d8xpg condition met
pod/alertmanager-77d4478976-24pgc condition met
pod/webhook-785c846b44-wxwwd condition met
pod/gateway-b768864ff-s54b2 condition met
```


3. Access gopaddle dashboard (Graphical User Interface) using the
above gopaddle access endpoint in a web browser of your choice.

The gopaddle lite access endpoint in the example shown above is:
```
http://127.0.0.1:30003
```

Equivalently, you can use the below as well:
```
http://localhost:30003
```

### Firewall settings

The following TCP network ports have to be opened by administrator for access:

- <b>Ports 30000 to 30006</b>: gopaddle-lite uses these TCP network ports to provide the gopaddle-lite access endpoints.

- <b>Port 16443</b>: The Kubernetes control plane on microk8s runs by default on this TCP network port
 
- <b>32000</b>: Service node port for Grafana dashboard on Kubernetes

- Any other TCP network port accessed by applications launched as Kubernetes services


### Access Modes

Access mode is http.  
https access is not supported in gopaddle Lite add-on.

Continuous Integration (CI) capability is not supported since that requires https access.

### Options supported by 'enable' in gopaddle lite ('-i' and '-v')

You can supply \<IP Address\> and \<gopaddle version\> through command line
options during 'enable' of gopaddle lite addon in microk8s.

Usage:
```
$ sudo microk8s enable gopaddle-lite -i <IP Address> -v <gopaddle version>
```

If '-i' option is used, the IP address specified is used as the static IP
address of the microk8s cluster. This could be the public or private IP address
configured by the administrator to access the gopaddle endpoint.

#### <i>Note: if '-i' and '-v' options are omitted, the default values used are as per the details already outlined under "Default values:" in section: "Steps to enable gopaddle addon for microk8s"</i>

Example:
```
$ sudo microk8s enable gopaddle-lite -i 130.198.9.42 -v 4.2.3
```

In this case, the 'enable' command will give the below output message:
```
Infer repository gp-lite for addon gopaddle-lite
static IP of the microk8s cluster: 130.198.9.42
...
Waiting for the gopaddle services to move to running state. This may take a while.
...
Enabling gopaddle lite
...
gopaddle lite is enabled

gopaddle lite access endpoint
http://130.198.9.42:30003
```

## Steps to disable gopaddle addon for microk8s

Issue the below command to disable gopaddle addon for microk8s:
```
$ sudo microk8s disable gopaddle-lite
```

You'll see the output as shown below:
```
Infer repository gp-lite for addon gopaddle-lite
Disabling gopaddle lite
...
namespace "gp-lite" deleted
Disabled gopaddle lite
```

## Steps to update gopaddle addon for microk8s

At a later time, if you want to update gopaddle addon repo (that you
previously added at the time of installation of gopaddle addon for microk8s),
use the below command:

```
$ sudo microk8s addons repo update gp-lite
```

This results in pulling any updates done to gopaddle addon repo. If it is
already up-to-date, you will get the below output:

```
Updating repository gp-lite
Already up to date.
```

If any new updates are pulled above, in order for this to take effect, you need to
execute the following steps:  
(1) Steps to disable gopaddle addon for microk8s (described in corresponding section above)  
(2) Steps to enable gopaddle addon for microk8s (described in corresponding section above)  
 

# Helm repository for gopaddle community (lite) edition

The 'enable' script above uses the Helm repository for gopaddle community (lite)
edition. The documentation for the same is available at: https://github.com/gopaddle-io/gopaddle-lite

# Support Matrix for gp-lite

The support Matrix for gopaddle lite 4.2.3 is located at:
http://help.gopaddle.io/en/articles/6227234-support-matrix-for-gopaddle-lite-4-2-3-community-edition
 
# Help

For help related to gopaddle community (lite) edition, visit the gopaddle Help Center at:
     https://help.gopaddle.io

Below is the command-line help for 'enable' command for gopaddle-lite 'addon' on microk8s:

```
$ sudo microk8s enable gopaddle-lite -i <IP Address> -v <gopaddle version>

Basic Options:
  --ip|-i      : static IP address to assign to gopaddle endpoint. This can be
                 a public or private IP address of the microk8s node
  --version|-v : gopaddle lite helm chart version
```

