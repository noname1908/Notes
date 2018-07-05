# Secrets and Configmaps

- Create secret
    ```sh
    kubectl create secrect generic tls-certs --from-file=tls/
    ```

- Describe secrets
    ```sh
    kubectl describe secrets tls-certs
    ```

- Create configmap
    ```sh
    kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
    ```

- Describe configmap
    ```sh
    kubectl describe configmap nginx-proxy-conf
    ```