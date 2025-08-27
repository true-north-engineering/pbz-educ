# Lab 1 - Openshift access:

Open OpenShift console. Login with your `IBM account`.

In top right corner, click on your `Username` and select `Copy login command`.

Open terminal and use `oc` to login into cluster.

Check your ID by running `oc whoami`.

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
  namespace: <your_username>
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

Use `oc` command to expose `todo-single` application.
Alternatively, you can use OpenShift web interface to do so.

Once deployment is exposed, find the service that was created using `oc` comand and in web interface.

# Lab 7 - Allow external acces for `todo-single` application with a route

Use either web console or oc command to create a route for `todo-single` application. The route should have the following hostname `todo-single-<userX>.ocp-edu.tn.hr`

Once you have the route created, note the url that was created.

# Lab 8 - Call `todo-single` application and correct the problem by adding env vars from secret

Test the application by opening the URL ```<OpenShift todo-single Route URl>/todo/```. Please note that the last trailing slash (/) is important, so don't omit him. 
Observe applications log and resolve the problems. 

Hint: Recall the lab1 and that the todo-single app needs certain environment variables which tells the application where the database is and how to connect to it, its credentails etc. Add these variables through OpenShift web interface. The database template which you used to create it also creates a secret with the connection parameters. Adjust the deployment so it uses this secret and the values in it for environment variables. Also, you will need to connect to the database pod and create the Items table.
```
mysql -h localhost -utodo -ptodopassw0rd todo
create table Item (id bigint not null auto_increment, description varchar(255), done bool, primary key (id));
```

# Lab 9 - Call `todo-single` application again and see if it works

# Lab 11 - Deployment strategies

Increase the replica count of `todo-single` app to 3. Then start new rollout of deployment. Observe pods and notice how for each new pod starting, an old one is destroyed.

Now change strategy to Recreate and start another rollot. Notice that this time all of the old pods get destroyed before new pods are created.

Scale the `todo-single` deployment back to 1 replicas.
