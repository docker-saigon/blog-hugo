+++
categories = ["Registry", "Docker-Machine"]
date = "2016-03-22T14:52:43+07:00"
description = "Creating a Private TLS secured Registry with Docker-Machine"
draft = false
image = "/img/private-registry.jpg"
tags = ["docker-machine","registry", "TLS", "Boot2Docker"]
title = "TLS secured Private Registries"
+++

An intermezzo after creating a small swarm cluster in [our previous post](/post/Swarm-Week-2016-Part1/) and before deploying & scaling the sample voting app on this cluster. 

In this post we will use `Docker-Machine` to provision a `Boot2Docker` host for a  local Docker `Registry` and explain how to configure other Machines (i.e. `Boot2Docker` & `CoreOS` machines) to push and pull from this Registry using TLS.

This setup is great for giving demonstrations where internet access is not guaranteed. We will not be configuring basic authentication or more advanced features such as web hooks, but provide links for how to do this.

We will, again, be using Windows 10, Hyper-V and PowerShell for this setup (as there aren't many guides out there using this setup), but it should be trivial to repeat these steps on OSX.

## 1. Create & Configure the Machine to host the Registry

Use Docker-Machine to create a TLS secured Docker Engine. For the full details on our environment configuration, refer to our previous post on [How to create a Swarm cluster.](/post/Swarm-Week-2016-Part1/)

As seen in that post, using Hyper-V and an elevated Powershell session with the `dm` alias (`New-Alias dm Docker-Machine`), we may run the following command to create our `registry0` machine:

```
dm create `
 --driver hyperv `
 --hyperv-virtual-switch "VMWare NAT" `
 --hyperv-memory "512" registry0
```

**Note**: As a result of the above command, `Docker-Machine` will have created a PKI for us. The Certificate Authority private key as well as self-signed CA certificate are stored under `~/.docker/machine/certs/`, we will use this information when generating the TLS assets for our registry.

Configure a static IP (192.168.233.3) for the newly created machine.

Using PowerShell with a single command from the Host:

```
echo "kill ``more /var/run/udhcpc.eth0.pid```n`
ifconfig eth0 192.168.233.3 netmask 255.255.255.0 broadcast 192.168.233.255 up`n`
route add default gw 192.168.233.2" | `
dm ssh registry0 "sudo tee /var/lib/boot2docker/bootsync.sh" > $null
```

(see Swarm post referred to earlier for a detailed explanation of these commands)

Now, bounce the machine:

```
dm restart registry0
```

After changing the IP, we have to re-generate the certificates used by our `registry0` machine:

```
dm regenerate-certs registry0
```

Add DNS entries for `registry0` (in our setup, this is handled through the `/etc/hosts` file on the VM Host):

```
"192.168.233.3 registry0`n192.168.233.3 registry0.localdomain" | ac $env:Windir\System32\Drivers\etc\hosts
```

**Note**: Requires an elevated PowerShell session.

## 2. Prepare the TLS assets for the Registry

For convenience, we will use the PKI created by `Docker-Machine` to generate a signed certificate.

First, generate the private key to be used by the registry server:

```
openssl genrsa -out registry-key.pem 2048
```

Next, we need to create a signing request. We will use a config file `registry-openssl.cnf` with the following contents:

```
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = registry0.localdomain
DNS.2 = registry0
IP.1 = 192.168.233.3
IP.2 = 192.168.150.104
```
**Note**: Make sure to replace/add any hostnames and IPs the registry will be reachable on by other Docker Engines into the above snippet.

Use the private key and the configuration file to create a certificate signing request (using git-bash):
```
openssl req -new -key registry-key.pem -out registry.csr -subj "/CN=registry0.localdomain" -config registry-openssl.cnf
```

Use the CA created by `Docker-Machine` to sign the certificate for the registry:

```
cp ~/.docker/machine/certs/ca*-pem .
openssl x509 -req -in registry.csr -CA "ca.pem" -CAkey "ca-key.pem" -CAcreateserial -out "registry.pem" -days 365 -extensions v3_req -extfile registry-openssl.cnf
```

Copy the registry private key, signed certificate as well as certificate authority to the registry server:

```
scp -i ~/.docker/machine/machines/registry0/id_rsa registry.pem registry-key.pem ca.pem docker@registry0:.
```

## 3. Create the Registry Container

We now have all ingredients to run a basic registry server on the `registry0` Machine. We will use a minimal Registry configuration, refer to the [official Docker docs](https://docs.docker.com/registry/configuration/) for more configuration options.

SSH to our Machine with `dm ssh registry0` or alternatively with the following command:

```
ssh -i ~/.docker/machine/machines/registry0/id_rsa docker@registry0
```

Prepare the folder structure and data for the registry server

```
sudo -i
mkdir /var/lib/boot2docker/registry-certs/
mkdir /var/lib/boot2docker/registry-data/
mv ~docker/registry-*.pem /var/lib/boot2docker/registry-certs/
```

Start the Registry container (this will pull the image automatically from the Docker Hub):
```
docker run -d -p 443:5000 --restart=always --name registry \
  -v /var/lib/boot2docker/registry-certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.pem \
  -e REGISTRY_HTTP_TLS_KEY=/certs/registry-key.pem \
  -v /var/lib/boot2docker/registry-data:/var/lib/registry \
  registry:2
```

**Note**: may need to chain the `registry.pem` certificate with the `ca.pem` certificate for this to work (to be confirmed)

From any other machine, confirm the registry works with the following command (assuming `ca.pem` lives in `PWD`)
```
curl --cacert ca.pem https://registry0/v2/
```

**Note**: To give an example of the more advanced configuration options, refer to [the configuration of webhooks](https://docs.docker.com/registry/configuration/#notifications): 

`REGISTRY_NOTIFICATIONS_ENDPOINTS_{name,url,headers, ...}` ENV variables which allow us to call out systems such as [Conduit](https://github.com/ehazlett/conduit) and automatically deploy images a build server may push to this registry.

![registry notifications](/img/notifications.png)

A more complicated setup may be managed through `Docker-Compose`.

## 4. Ensuring your Docker Engine can push/pull from this registry. 

Servers who do not trust the CA which signed the Registry certificate will not be able to push/pull from our Registry. The steps below show how to make a server trust our CA.

### For Boot2Docker machines:

Add the Certificate Authority to a `Boot2Docker` machine (See: [B2d - Installing Secure Registry Certificates](https://github.com/boot2docker/boot2docker/blob/v1.10.3/README.md#installing-secure-registry-certificates)):

```
dm scp ca.pem <machine>:~
dm ssh <machine> sudo mkdir -p /var/lib/boot2docker/certs/
dm ssh <machine> sudo mv ~/ca.pem /var/lib/boot2docker/certs/
dm restart <machine>
```

Once the machine has rebooted, you will be able to push/pull from the local registry.

### For CoreOS machines:

Similar to Boot2Docker, add the Certificate Authority and update the certificates:

```
scp ca.pem core@<machine>:~
ssh core@<machine> sudo mv ca.pem /etc/ssl/certs
ssh core@<machine> sudo update-ca-certificates
```

### For Ubuntu, Debian, RHEL, CentOS, ...

Refer to the [official documentation](https://docs.docker.com/docker-trusted-registry/configure/config-security/#install-registry-certificates-on-client-docker-daemons) from the DTR (the setup is the same for our TLS secured private registry).

## 5. Usage tips for your private Registry

To pull a Docker Image from the Hub and make it available on your local registry, enter the following commands (example with the official alpine-based nginx image):

```
docker pull nginx:mainline-alpine
docker tag nginx:mainline-alpine registry0.localdomain/nginx:mainline-alpine
docker push registry0.localdomain/nginx:mainline-alpine
```

From then on, all local machines may easily serve static content with the following command:

```
docker run -d --name web -v --restart=always /path/to/html:/etc/nginx/html:ro registry0.localdomain/nginx:mainline-alpine
```

**Tip**: this nginx server can be deployed on our registry server to serve static binaries / yaml files for bootstrapping scripts for our cluster as well as the images stored in its repositories.

There are a few Registry web UI which allow you to list repositories, images and tags for a v2 Registry, but several lack basic features.

Alternatively, the contents may be listed with the following `curl` & `jq` commands as well:

```
#list repositories
curl -s https://registry0.localdomain/v2/_catalog | jq -r .repositories[]
#list tags of an image
curl -s https://registry0.localdomain/v2/nginx/tags/list | jq -r .tags[]
```

For more actions, refer to the [Registry v2 API](https://github.com/docker/distribution/blob/v2.3.1/docs/spec/api.md#deleting-an-image)