
# ConfigMap

Resources that Kubernetes provides for configuring Pods.

## What is ConfigMap and when is it used? 

Think of it as a properties file for your application. For example depending on your application environment (dev, int, prod) you will have a different database url or logging level. So for these kind of things you can use configMap.

The biggest advantage is that, with properties file, every time you modify it you have to rebuild and redeploy your application, whereas if you change configuration in configMap, you just need to restart the application pod/container.

ConfigMap can be used by the application as a set of environmental variable values or as an actual configuration file.

Example ConfigMap with database connection configuration:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: my-config
data:
  db-host: cluster-mysql.database 
  db-port: 3306
  db-name: my-db 
```

The values in this configMap can be used in a following way in your app's pod specification:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-app
    image: my-app-image
    env:
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: db-host
    - name: DB_PORT
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: db-port
    - name me: DB_NAME
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: db-name
```

Here is an example ConfigMap which creates a configuration file for Mosquitto app:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: mosquitto-config
data:
  mosquitto.conf |
     log_dest stout
     log_type all
     log_timestamp true
     listener 9001
```

In this case we need to mount the ConfigMap as a volume in Kubernetes:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mosquitto
spec:
  containers:
  - name: mosquitto
    image: mosquitto-image
    volumeMounts:
      - name: config-file
        mountPath: /mosquitto/config
  volumes:
  - name: config-file
    configMap:
      name: mosquitto-config
```

This config map will produce a file mosquitto.conf, which then can be mounted into the Mosquitto container under /mosquitto/config directory.

##Secret

Secrets 🔐 are also used in these 2 ways. Either as a value for env variables or as a secret file with credentials or a certificate etc mounted into a pod.

So for a better comparison, think of secrets as encrypted configMaps.

Example secret with key-value pairs:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  type: Opaque
data:
  db-user: dXNlcg==
  db-password: cGFzc3dvcmQ
```

And you can use it the same way as ConfigMap in your application's configuration file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-app
    image: my-app-image
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: db-user
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: db-password
```

Here is an example secret that creates a file:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  type: Opaque
data:
  cacert.pem |
     base-64-encoded value of a PEM certificate
```

And again, just like with ConfigMap, you will need to mount this secret as a volume into the pod to use the cacert.pem file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-app
    image: my-app-image
    volumeMounts:
      - name: certificate-file
        mountPath: /etc/secret
  volumes:
  - name: certificate-file
    configMap:
      name: my-secret
```

The inconvenience with this way of creating a secret for a file is that you will have to base64 encode the file contents and then paste it into the data section.

So an easier alternative way to create secrets from a file is with kubectl command.

Like in the above case, get the cacert.pem file and execute:

```yaml
kubectl create secret generic my-secret --from-file=./cacert.pem
```

- __ConfigMap__ is for non-confidential data only
- __Secret__ is for confidential data only
- __Secret__ stored data in base64 format ```echo -n password | base64```
- configMap and secret are accessible via env variables or properties file
