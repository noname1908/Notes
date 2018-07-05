# Google Kubernetes Engine Guestbook Tutorial

## sample repository
```sh
git clone https://github.com/GoogleCloudPlatform/kubernetes-engine-samples.git
cd kubernetes-engine-samples/guestbook
```
## Configuring your deployment
- Exploring the controller
    ```sh
    cat redis-master-controller.yaml
    ```
    This file contains configuration to deploy a Redis master.
    - Kind:
    
        Although we have a single instance of our redis master, we are using a Replication Controller to enforce that exactly one pod keeps running. E.g., if the node were to go down, the replication controller will ensure that the redis master gets restarted on a healthy node. (In our simplified example, this could result in data loss.)

    - Metadata:

        The **metadata**: sections define labels to apply to the Replication Controller and related Pods. Labels are simple key-value pairs, which can be queried by other operations.

    - Spec:

        Here we define the Pod specification which the Replication Controller will use to create the Redis pod. The **image**: tag refers to a Docker image to be pulled from a registry.

- Exploring your service
    ```sh
    cat redis-master-service.yaml
    ```

    This file contains configuration to deploy a Service for the Redis master. A Kubernetes service is a named load balancer that proxies traffic to one or more containers. This is done using the labels metadata that we defined in the **redis-master** pod. As mentioned, we have only one Redis master, but we nevertheless want to create a service for it. Why? Because it gives us a deterministic way to route to the single master using an elastic IP.

    Services find the pods to load balance based on the pods' labels. The **selector**: field of the service description determines which pods will receive the traffic sent to the service, and the **port**: and **targetPort**: information defines what port the service proxy will run at.

## Deploy the Redis Master
- Setup gcloud and kubectl credentials
    ```sh
    gcloud container clusters get-credentials cluster-1 --zone us-central1-a
    ```

- Create a Service

    Create a service before corresponding replication controllers so that the scheduler can spread the pods comprising the service. So we first create the service by running:
    ```sh
    kubectl create -f redis-master-service.yaml
    ```

- Create a Replication Controller

    ```sh
    kubectl create -f redis-master-controller.yaml
    ```

- Inspect your cluster

    List the pods, replication controllers, and services by running:

    ```sh
    kubectl get pods
    kubectl get rc
    kubectl get services
    ```

## Deploy the Redis Read Replicas

The Redis read replicas contain similar configuration in the guestbook/ directory. This time, we will use a single .yaml file to deploy both the Service and Replication Controller. The Replication Controller configuration contains replicas: 2 which will create 2 pods.

- Deploy

    Deploy the Service and Replication Controllers by running:
    ```sh
    kubectl create -f all-in-one/redis-slave.yaml
    ```

- Inspect your cluster

    List the pods, replication controllers, and services by running:

    ```sh
    kubectl get pods
    kubectl get rc
    kubectl get services
    ```

## Deploy the Guestbook Frontend

A frontend pod is a simple PHP server that is configured to talk to either the slave or master services, depending on whether the client request is a read or a write. It exposes a simple AJAX interface, and serves an Angular-based UX. Again we'll create a set of replicated frontend pods instantiated by a replication controller— this time, with three replicas.

As with the other pods, we now want to create a service to group the frontend pods. The RC and service are described in the file frontend.yaml.

- Deploy

    Deploy the Service and Replication Controllers by running:
    ```sh
    kubectl create -f all-in-one/frontend.yaml
    ```

- Find your external IP

    List the services, and look for the frontend service. Wait until you see an IP show up under the External IP column. This may take a minute. Afterwards you can stop monitoring by pressing Ctrl+C.
    ```sh
    kubectl get services --watch
    ```

- Visit your running Guestbook app

    Copy the IP in the External IP column.

    Open a new tab and visit your Guestbook app, by navigating to the IP.

- Delete your cluster

    With a Kubernetes Engine cluster running, you can create and delete resources with the kubectl command-line client. When you are done, you will need to shut down the entire cluster to avoid subsequent charges.

    - Click the Trash Can

        Click the trash can next to the cluster we created cluster-1. Then click Delete.