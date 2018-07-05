# Deploying Ambassador to Kubernetes

1. [Create Kubernetes Engine](https://console.cloud.google.com/kubernetes)
2. Deploying Ambassador RBAC is enabled
    ```sh
    kubectl create clusterrolebinding my-cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud info --format="value(config.account)")
    ```
    ```sh
    kubectl apply -f https://getambassador.io/yaml/ambassador/ambassador-rbac.yaml
    ```
3. Defining the Ambassador Service
    ```sh
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: ambassador
    spec:
      type: LoadBalancer
      ports:
      - port: 80
      selector:
        service: ambassador
    ```
    Deploy this service with kubectl:
    ```sh
    kubectl apply -f ambassador-service.yaml
    ```
4. Testing the Mapping
    ```sh
    kubectl get svc -o wide ambassador
    ```
5. Adding a Service
    ```sh
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: qotm
      annotations:
        getambassador.io/config: |
          ---
          apiVersion: ambassador/v0
          kind:  Mapping
          name:  qotm_mapping
          prefix: /qotm/
          service: qotm
    spec:
      selector:
        app: qotm
      ports:
      - port: 80
        name: http-qotm
        targetPort: http-api
    ---
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: qotm
    spec:
      replicas: 1
      strategy:
        type: RollingUpdate
      template:
        metadata:
          labels:
            app: qotm
        spec:
          containers:
          - name: qotm
            image: datawire/qotm:1.1
            ports:
            - name: http-api
              containerPort: 5000
            resources:
            limits:
              cpu: "0.1"
              memory: 100Mi
    ```
    ```sh
    kubectl apply -f qotm.yaml
    ```
6. The Diagnostics Service in Kubernetes
    ```sh
    $ kubectl get pods
    NAME                          READY     STATUS    RESTARTS   AGE
    ambassador-3655608000-43x86   1/1       Running   0          2m
    ambassador-3655608000-w63zf   1/1       Running   0          2m
    ```
    Forwarding local port 8877 to one of the pods:
    ```sh
    kubectl port-forward ambassador-3655608000-43x86 8877
    ```