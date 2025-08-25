# Lab 1 - Openshift access:

Open OpenShift console. Login with your `IBM account`.

In top right corner, click on your `Username` and select `Copy login command`.

Open terminal and use `oc` to login into cluster.

Check your ID by running `oc whami`.

List all projects.

# Lab 2 - Create Project

Create a project for rest of the labs. The name of the project must be your designated `userX` username.

Use either Openshift web console, or `oc` command to create a new project named `UserX`, same as your user on day 1.

# Lab 3 - Deploy `MySQL (Ephemeral)` database from template.

Login to OpenShift web console.

Choose the `Developer` console from top left dropdown.

Add `MySQL (Ephemeral)` database to your project.

Set `Database Service Name` to `todo-db`
Set `MySQL Connection Username` to `todo`
Set `MySQL Connection Password` to `todopassw0rd`
Set `MySQL Database Name` to `todo`

Leave rest of the fields with default values.

Wait for `MySQL (Ephemeral)` to be ready.

# Lab 4 - Create image pull secret

Login to OpenShift console.

Select secrets in ether administrator console under workloads, or developer console at bottom of menu.

Choose `Create` and select `Image pull secret` from dropdown menu.

Use following parameters
```
name: nexus-pull-secret
url: docker-nexus-edu.tn.hr
user: user
password: XXX
```

# Lab 5 - Deploy `todo` application using `oc` command

`animals` is an aplication that tells you what animals live on your servers. When deploying app you can choose what animal you want to check by setting `ANIMAL` environment variable, and if that animal exists, it will tell you their names.

To know whether animal exists, it checks in PostgreSQL database you just created. Let's check if there are any cats.

To deploy application, create a deployment file using following template:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: <namespace>
  name: <name>
spec:
  selector:
    matchLabels:
      app: <name>
  replicas: 1
  template:
    metadata:
      labels:
        app: <name>
    spec:
      containers:
        - name: <name>
          image: <image>
          ports:
            - containerPort: <port>
              protocol: TCP
          env:
            - name: ENV_VAR
              value: value
      imagePullSecrets:
        - name: nexus-pull-secret
```
Use following data to complete the config:
```
name: cat
port: 5000
image: docker-nexus-edu.tn.hr/animals:latest
env:
  ANIMAL: cat
  DB_HOST: animalsdb
  DB_PORT: 5432
  DB_USER: animal
  DB_PASSWORD: 4n1m4l!
  DB_NAME: animals
```

Create the deployment using oc command:
```
$ oc apply -f animals-cat-deployment.yaml
```

# Lab 6 - Expose `animals` application

Use `oc` command to expose `animals` application.

Once deployment is exposed, find the service that was created using `oc` comand and in web interface.

# Lab 7 - Allow external acces for `animals` application with a route

Use either web console or oc command to create a route for `animals` application.

Once you have the route created, note the url that was created.

# Lab 8 - Call `animals` application

Open the url from `animals` route in your browser and see the result. You should get a result similar to:
```
It's a cat named Sundance living on cat-d77c74f76-rnqpd.
```
Each time you call the service you get a different name. The `cat-d77c74f76-rnqpd` in exmaple is the name of the pod that served the request.

You can do the same thing in console using curl:
```
$ curl http://animals-userX.ocp.pbz.tn.hr/
It's a cat named Lucy living on cat-d77c74f76-rnqpd.
```

# Lab 9 - Add environmet variables from secret

When you deployed your `PostgreSQL` database, a `secret` with the same name was created. Inside that secret are parameters to connect to DB.

Open the deployment config of `animals` application.

Under environment tab replace static environment variables with those from `PostgreSQL` secret.

Compare the new `yaml` configuration with your initial deployment file.

# Lab 10 - Add configurations from configmap

Now that we have database config from secret, externalize rest of the config by creating a config map.

Add following values to configmap:
```
ANIMAL
DB_HOST
DB_PORT
```
Add the values from configmap to `animals` application pod.

# Lab 11 - Deployment strategies

Increase the replica count of `animals` app to 3. Then start new rollout of deployment. Observe pods and notice how for each new pod starting, an old one is destroyed.

Now change strategy to Recreate and start another rollot. Notice that this time all of the old pods get destroyed before new pods are created.

Scale the `cat` pod replicas to 1.

# Lab 12 - Restart postgresql database

Restart the database by deleting it's pod.

Try calling `animals` app and see the result.

The database we created is `ephemeral` so every time we restart the database, we loose all the data.

# Lab 13 - Create a persistent volume claim

Each time `animals` app is started it creates the db if it's missing and populates it with names from `/app/names' file inside container. We will replace that file with our own list of names and repopulate the database.

Create a `Persistent Volume Claim` named `animal-data`. Use `ibmc-file-bronze-gid` storage class and `rwx` access mode, the volume can be small, 100MiB is more than enough.

# Lab 14 - Populate volume with data

Each time `animals` app is started it creates the db if it's missing and populates it with names from `/app/data/names` file inside container. We will replace that file with our own list of names and repopulate the database. A file has been pepared for you in github repo at `https://raw.githubusercontent.com/true-north-engineering/pbz-educ/refs/heads/main/lab3/names`, but you can use your own.

Create a job that uses `alpine/curl` image to download the names file into `PVC` you created before.

# Lab 15 - Mount the volume into animals container

Mount the volume with your custom names into `animals` app pod under `/app/data/` path. Changing deployment will automatically initiate restarting of pod. Once restarted try calling the app again and see that names are now those in the new file.
