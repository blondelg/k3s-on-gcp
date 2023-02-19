# k3s-on-gcp
The aim of this repo is to provide a memo to build a 3 nodes [k3s](https://docs.k3s.io/) cluster on three google cloud e2 instances. I don't recommand to use it for production purposes but for learning or development.

## 1 - Google cloud setup
### a. VPC setup
First of all, a VPC network should be created.
On the firewall, I have created the following rules:
* **ssh**, change ingress tcp traffic to a random port (Port has to be updated on `/etc/ssh/sshd_config` file of each node, then ssh service restarted `systemctl restart ssh`)
* **local egress**, allow all egress on the local ip range (for instance 10.2.0.0/24)
* **local ingress**, allow all ingress on the local ip range (for instance 10.2.0.0/24)
### b. Instances setup
I have created 3 isntances, one master and two agents. All of them are plugged to the previousply created network.
#### Install on master
```
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" K3S_TOKEN=my-safe-secret sh -s - server --cluster-init
```
#### Install on agents
```
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" K3S_TOKEN=my-safe-secret sh -s - server --server https://<ip or hostname of master>:6443
```

### c. GCR setup
To host imges, I use Google Container Registry. The following steps have to be done:

* Create a service account with cloud storage role like **Storage Object Viewer**.
* On service account, go to keys and add a json key.
* On each node, update this file **/etc/rancher/k3s/registries.yaml** and populate as follow (*password* is the json key generated on the previous step):
```
mirrors:
  eu.gcr.io:
    endpoint:
      - "https://eu.gcr.io"
configs:
  "eu.gcr.io":
    auth:
     username: _json_key
     password: '{  "type": "service_account", "project_id": .....}'
```
* Restart k3s `sudo systemctl restart k3s`
