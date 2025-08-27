# Lab1 - Openshift access:

Open OpenShift console. Login with your `IBM account`.

In top right corner, click on your `Username` and select `Copy login command`.

Open terminal and use `oc` to login into cluster.

Check your ID by running `oc whami`.

List all visible projects.
```
$ oc projects
```

# Lab2 - Create Project

Create a project for rest of the labs.

Use either Openshift web console, or `oc` command to create a new project named `userX`, same as your user on day 1.

```
$ oc new-project userX
```

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

Run:
```
$ oc get pods
```
And wait for output similar to:
```
NAME                   READY   STATUS      RESTARTS   AGE
todo-db-1-7vlq4        1/1     Running     0          3m
```
`animalsdb` pod should be in `Runnning` state.


# Lab 4 - Create image pull secret

Login to OpenShift console.

Select secrets in either the administrator console under workloads, or developer console at the bottom of menu.

Choose `Create` and select `Image pull secret` from dropdown menu.

Use following parameters
```
name: nexus-pull-secret
url: docker-nexus-edu.tn.hr
user: user
password: XXX
```

# Lab 5 - Deploy `todo-single` application using `oc` command or through OpenShift Web Interface


To deploy the application through oc command, create a deployment file using template below.
If you are using OpenShift web interface, use the values from the template in appropriate fields.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: user50
  name: todo-single
spec:
  selector:
    matchLabels:
      app: todo-single
  replicas: 1
  template:
    metadata:
      labels:
        app: todo-single
    spec:
      containers:
        - name: todo
          image: docker-nexus-edu.tn.hr/edu/todo-single:latest
          ports:
            - containerPort: 30080
              protocol: TCP
      imagePullSecrets:
        - name: nexus-pull-secret
```

Create the deployment using oc command:
```
$ oc apply -f <file>
```

# Lab 6 - Expose `todo-single` application

Use `oc` command to expose `todo-single` application. Alternatively, you can use OpenShift web interface to do so.

Once deployment is exposed, find the service that was created using `oc` comand and in web interface.

```
$ oc expose deployment todo-single
```
Once deployment is exposed, find the service that was created using `oc` comand and in web interface.
```
$ oc get services
```

# Lab 7 - Allow external acces for `todo-single` application with a route

Use either web console or oc command to create a route for `todo-single` application. The route should have the following hostname `todo-single-<userX>.ocp-edu.tn.hr`

Once you have the route created, note the url that was created.

```
$ oc expose service todo-single --hostname todo-single-user50.ocp-edu.tn.hr
```
Once you have the route created, note the url that was created.
```
$ oc get routes
```

# Lab 8 - Call `todo-single` application and correct the problem by adding env vars from secret

Test the application by opening the URL ```todo-single-user50.ocp-edu.tn.hr/todo/```. Please note that the last trailing slash (/) is important, so don't omit him. 
Observe applications log and resolve the problems. 

Hint: Recall the lab1 and that the todo-single app needs certain environment variables which tells the application where the database is and how to connect to it, its credentails etc. Add these variables through OpenShift web interface. The database template which you used to create it alse creates a secret with the connection parameters. Adjust the deployment so it uses this secret and the values in it for environment variables. Also, you will need to connect to the database pod and create Items table.
```
mysql -h localhost -utodo -ptodopassw0rd todo
create table Item (id bigint not null auto_increment, description varchar(255), done bool, primary key (id));
```

To add env vars modify the todo-single deployment yaml and add the `env` section:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: user50
  name: todo-single
spec:
  selector:
    matchLabels:
      app: todo-single
  replicas: 1
  template:
    metadata:
      labels:
        app: todo-single
    spec:
      containers:
        - name: todo
          image: docker-nexus-edu.tn.hr/edu/todo-single:latest
          ports:
            - containerPort: 30080
              protocol: TCP
          env:
            - name: MYSQL_ENV_MYSQL_HOST
              value: todo-db
            - name: MYSQL_ENV_MYSQL_PORT
              value: '3306'
            - name: MYSQL_ENV_MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: todo-db
                  key: database-name
            - name: MYSQL_ENV_MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: todo-db
                  key: database-user
            - name: MYSQL_ENV_MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: todo-db
                  key: database-password
      imagePullSecrets:
        - name: nexus-pull-secret
```

# Lab 9 - Call `todo-single` application again and see if it works

# Lab 10 - Deployment strategies

Increase the replica count of `todo-single` app to 3. Then start new rollout of deployment. Observe pods and notice how for each new pod starting, an old one is destroyed.

Now change strategy to Recreate and start another rollot. Notice that this time all of the old pods get destroyed before new pods are created.

Scale the `todo-single` deployment back to 1 replicas.

```
$ oc scale deployment todo-single --replicas 3
```
Then start new rollout of deployment.
```
$ oc rollout restart deployment todo-single
```
Observe pods and notice how for each new pod starting, an old one is destroyed.

Now change strategy to Recreate.
Edit the `todo-single` deployment:
```
$ oc edit deployment todo-single
```
Replace:
```
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
```
With:
```
  strategy:
    type: Recreate
```
Start another rollout.
```
$ oc rollout restart deployment todo-single
```
Notice that this time all of the old pods get destroyed before new pods are created.

Scale the `todo-single` pod replicas to 1.
```
$ oc scale deployment todo-single --replicas 1
```
