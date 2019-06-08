> So Kubernetes is pronounced /koo-ber-nay'-tace/ and [means "sailing master"][kube-meaning]

Via [@francesc][kube-twitter]

## Terminology

- **Node**: A single virtual or physical machine in a Kubernetes cluster.
- **Cluster:** A group of nodes firewalled from the internet, that are the primary compute resources managed by Kubernetes.
- **Edge router:** A router that enforces the firewall policy for your cluster. This could be a gateway managed by a cloud provider or a physical piece of hardware.
- **Cluster network:** A set of links, logical or physical, that facilitate communication within a cluster according to the [Kubernetes networking model][kub-networking]. Examples of a Cluster network include Overlays such as flannel or SDNs such as OVS.
- **Service:** A Kubernetes Service that identifies a set of pods using label selectors. Unless mentioned otherwise, Services are assumed to have virtual IPs only routable within the cluster network.

### What is a pod??

A **pod is a group of one or more containers** (such as Docker containers), with shared storage/network, and a specification for how to run the containers. A pod’s contents are always co-located and co-scheduled, and run in a shared context.

Containers within a pod share an IP address and port space, and can find each other via localhost. They can also communicate with each other using standard inter-process communications like SystemV semaphores or POSIX shared memory. Containers in different pods have distinct IP addresses and can not communicate by IPC without special configuration. These containers usually communicate with each other via Pod IP addresses.

In terms of Docker constructs, **a pod is modelled as a group of Docker containers with shared namespaces and shared volumes**.

### Kubernetes "plugins"

- [Helm Tiller](https://docs.helm.sh/): Helm is a package manager for Kubernetes and is required to install all the other applications. It is installed in its own pod inside the cluster which can run the `helm` CLI in a safe environment.
- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/): Ingress can provide load balancing, SSL termination, and name-based virtual hosting. It acts as a web proxy for your applications and is useful if you want to use [Auto DevOps](../../../topics/autodevops/index.md) or deploy your own web apps.
- [Prometheus](https://prometheus.io/docs/introduction/overview/): Prometheus is an open-source monitoring and alerting system useful to supervise your deployed applications.

## How to manage the Google Kubernetes Engine
The first step is to check the defined clusters [using the Google Cloud Platform console][gcp-console]. The most common target is, at first, to build and push a containarised application, which means that we start creating a new docker image in our machine, then after checking it locally, we push the image to a *docker container registry*. 

Using GCP all these operations can be done by using a mix of `docker`, `gcloud` and `kubectl` commands. First things first, we must create one or more images using the instructions provided [here](Docker). After that, one can follow the instructions of [this gcloud tutorial][gcloud-tutorial].

### Setting a static IP and a custom domain for the application

In our case we can [assign a static IP address][kub-ip-service] for the service created one step above. The best way to achieve this, IMHO, is the web interface and precisely [here](https://console.cloud.google.com/networking/addresses/add?project=together-rx). 

### Update the image used for the current Kubernetes POD

1. Build a new snapshot optimized for production
`ng build --prod`

2. Create a new image and tag it with a new version

3. Test the new image locally

3. Push the newly created image to the container registry

To update the deployed container we can use the set command:
`kubectl set image deployment/together-rx ...`

**Problem:** A frequent question that comes up on Slack and Stack Overflow is how to trigger an update to a Deployment/RS/RC when the image tag hasn't changed but the underlying image has.

Consider:

1. There is an existing Deployment with image `foo:latest`
2. User builds a new image `foo:latest`
3. User pushes `foo:latest` to their registry
4. User wants to do something here to tell the Deployment to pull the new image and do a rolling-update of existing pods

**Possible solution for our current scenario:** you are using `'latest'` for testing (this is the "no sed" use case), in this case, downtime is fine, and indeed the right approach is likely to *completely blow away your stack* and redeploy from scratch to get a clean run (=> `kubectl set <...>`).


Keep reading [here][kubernetes-redeploy].

### Delete a Kubernetes service

To stop and delete a Kubernetes instance/service corresponding to the image we created and then exposed we should instruct Kubernetes to tell the Load Balancer to delete the provisioned service with a simple command like that:

`kubectl delete service container-1`

**NOTE:** The load balancer is deleted asynchronously in the background when you run `kubectl delete`. Wait until the load balancer is deleted by watching the output of the following command:

`gcloud compute forwarding-rules list`

### Storage and DBs on GKE

- [High Availability PostgreSQL and Kubernetes with Google Cloud](https://codelabs.developers.google.com/codelabs/cloud-postgresql-gke-memegen/#0)


- [How to pronounce Kubernetes][kubeforvo]

[kubeforvo]: https://forvo.com/search/Kubernetes/
[kube-meaning]: https://www.biblestudytools.com/lexicons/greek/nas/kubernetes.html
[kube-twitter]: https://twitter.com/francesc/status/487412202932936704

[gcloud-tutorial]: https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app
[kubernetes-redeploy]: https://github.com/kubernetes/kubernetes/issues/33664#issuecomment-258349316
[kub-networking]: https://kubernetes.io/docs/concepts/cluster-administration/networking/
[kub-ip-service]: https://cloud.google.com/kubernetes-engine/docs/tutorials/configuring-domain-name-static-ip#step_2a_using_a_service