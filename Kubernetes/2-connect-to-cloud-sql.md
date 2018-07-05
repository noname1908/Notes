# Connect to Cloud SQL instance, using the Cloud SQL Proxy Docker image.

1. Enable the API
    - [Enable the Cloud SQL Administration API.](https://console.cloud.google.com/flows/enableapi?apiid=sqladmin)
2. Create a service account
    - [Go to the service account page](https://console.cloud.google.com/iam-admin/serviceaccounts)
    - If needed, select the project that contains your Cloud SQL instance.
    - Click **Create service account**
    - Provide a descriptive name for the service account
    - In **Role**, select **one** of the following roles:
        - **Cloud SQL > Cloud SQL Client**
        - **Cloud SQL > Cloud SQL Editor**
        - **Cloud SQL > Cloud SQL Admin**
    - Change the **Service account ID** to a unique value
    - Click **Furnish a new private key**
    - The default key type is **JSON**
    - Click **Create** and download the private key file (`PROXY_KEY_FILE_PATH`)
3. Create the proxy user
    ```sh
    gcloud sql users create proxyuser cloudsqlproxy~% --instance=[INSTANCE_NAME] --password=[PASSWORD]
    ```
4. Get your instance connection name (`INSTANCE_CONNECTION_NAME`)
    ```sh
    gcloud sql instances describe [INSTANCE_NAME]

    // connectionName: ...
    ```
5. Create your Secrets
    - The `cloudsql-instance-credentials` Secret contains the service account.
        ```sh
        kubectl create secret generic cloudsql-instance-credentials \
            --from-file=credentials.json=[PROXY_KEY_FILE_PATH]
        ```
    - The `cloudsql-db-credentials` Secret provides the proxy user account and password.
        ```sh
        kubectl create secret generic cloudsql-db-credentials \
            --from-literal=username=proxyuser --from-literal=password=[PASSWORD]
        ```
6. Update your pod configuration file
    ```sh
    ...
    spec:
      ...
      spec:
        containers:
        - name: ...
          env:
            - name: WORDPRESS_DB_HOST
              value: 127.0.0.1:3306
            # These secrets are required to start the pod.
            # [START cloudsql_secrets]
            - name: WORDPRESS_DB_USER
              valueFrom:
                secretKeyRef:
                  name: cloudsql-db-credentials
                  key: username
            - name: WORDPRESS_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: cloudsql-db-credentials
                  key: password
            # [END cloudsql_secrets]
        # Change <INSTANCE_CONNECTION_NAME> here to include your GCP
        # project, the region of your Cloud SQL instance and the name
        # of your Cloud SQL instance. The format is
        # $PROJECT:$REGION:$INSTANCE
        # [START proxy_container]
        - name: cloudsql-proxy
          image: gcr.io/cloudsql-docker/gce-proxy:1.11
          command: ["/cloud_sql_proxy",
                    "-instances=<INSTANCE_CONNECTION_NAME>=tcp:3306",
                    "-credential_file=/secrets/cloudsql/credentials.json"]
          volumeMounts:
            - name: cloudsql-instance-credentials
              mountPath: /secrets/cloudsql
              readOnly: true
        # [END proxy_container]
      # [START volumes]
      volumes:
        - name: cloudsql-instance-credentials
          secret:
            secretName: cloudsql-instance-credentials
      # [END volumes]
    ```
    ```sh
    kubectl apply -f deployment.yaml
    ```
7. Update your application to connect to Cloud SQL

    When you have your Kubernetes Engine environment set up, you connect to Cloud SQL the same way as any other external application that is using the proxy. The exact connection string you use depends on what language and framework you are using.