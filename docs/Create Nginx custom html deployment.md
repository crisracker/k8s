# Create Nginx with custom html deployment

## Create a Nginx deployment

```cmd

kubectl create deployment ct-nginx --image=nginx --replicas=3 --port=80

```

## Create a NodePort service to the deployment

We will add a NodePort service which will allow traffic from outside into the K8s cluster. It provides a layer of abstraction and allows the client to communicate with the service without knowledge of the application running on multiple pods. The following command will expose the deployment externally.

```cmd

kubectl expose deployment ct-nginx --port=80 --type=NodePort

```

To test, we can run Curl or just open up an browser and navigate to the IP:Port 

```cmd

curl “IP address:Port”

```

Use the command below to view the replica sets

```cmd

kubectl get rs

```

## Create a ConfigMap

The next step is to create a ConfigMap. We will generate a ConfigMap file that will contain our custom HTML. The example below shows our completed file.

```cmd

apiVersion: v1
kind: ConfigMap
metadata:
  name: ct-html-configmap
  namespace: default
data:
  index.html: |
    <html>
    <h1>Welcome to CT Page<h1>
    </br>
    <h1>This is CT web. Running on a Pod running Nginx.<h1>
    </html>

```

Create the ConfigMap

```cmd

kubectl apply -f nginx-configmap

```

Now to edit the deployments

```cmd

kubectl edit deployment ct-nginx

```

## Attach the configMap to the deployment

```cmd

apiVersion: apps/v1
kind: Deployment
metadata:
  name: ct-nginx
  namespace: default
  labels: 
    name: ct-nginx
spec:
  selector:
    matchLabels:
      name: ct-nginx
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: ct-nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
          volumeMounts:
          - name: nginx-index-file
            mountPath: /usr/share/nginx/html
      volumes:
        - name: nginx-index-file
          configMap:
            # Provide the name of the ConfigMap containing the files you want
            # to add to the container
            name: nginx-index-file

```

The below is added to the deployment.

```cmd

spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        volumeMounts:
          - name: nginx-index-file
            mountPath: /usr/share/nginx/html
      volumes:
      - name: nginx-index-file
        configMap:
          name: ct-html-configmap

```

```

#1
volumeMounts:
          - name: nginx-index-file
            mountPath: /usr/share/nginx/html
#2
volumes:
      - name: nginx-index-file
        configMap:
          name: ct-html-configmap

```

Refresh the webpage and you should see the custom page
