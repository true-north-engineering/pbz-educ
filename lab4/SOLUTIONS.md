# Lab 1 - CI, CD and GitOps

Before starting with lab exercises connect to the linux system box-edu.tn.hr with ssh or putty as userX where X is a number designated to you by the presenter.
Example:

```ssh userX@box-edu.tn.hr```

## Task 1 - S2I build

1. Clone the pbz-educ-src in your home folder.

```git clone https://github.com/true-north-engineering/pbz-educ-src.git```

2. Generate Containerfile for source in ```pbz-educ-src/todo-frontend``` based on the builder image ```registry.access.redhat.com/ubi8/nodejs-16:latest```

```
cd ~/pbz-educ-src/todo-frontend
s2i build --as-dockerfile ../Containerfile.gen . registry.access.redhat.com/ubi8/nodejs-16:latest
```

3. Build the container image from the generated Containerdile

```
cd ~/pbz-educ-src
podman build -t todo-frontend:latest -f Containerfile.gen
```

## Task 2 - Tekton builds

1. Create an Openshift project called by your username, e.g. user1

```
oc new-project user1
oc project user1
```

2. Create ```nexus-docker-auth``` secret that containes authentication data for ```docker-nexus-edu.tn.hr```. Link that secret with ```pipeline``` service account.

```
Create nexus-docker-auth secret in user1 project through Openshift web interface or CLI:
Name: nexus-docker-auth
Registry server address: docker-nexus-edu.tn.hr
Username: user
Password: <nexus_password>

Open the pipeline ServiceAccount and under secrets add 
  - name: nexus-docker-auth

or CLI

oc create secret docker-registry nexus-docker-auth --docker-server=docker-nexus-edu.tn.hr --docker-username=user --docker-password=<nexus_password>
oc secret link pipeline nexus-docker-auth
```

2. Connect to the Openshift console. In project ```pbz-cicd``` you have Tekton pipeline examples. Copy them to your project.

3. Create todo-api pipeline which has the following properties:
    * Define one parameter ```revision``` which will hold the git branch or commit sha of the code to clone.
    * Define one workspace ```source``` which represents the workspace (persistent volume) where the source code will be cloned locally.
    * First task is git-clone:
        * Github URL to clone is ```https://github.com/true-north-engineering/pbz-educ-src.git```
        * Revision to checkout is taken from the ```revision``` parameter. Hint: ```$(params.revision)```
        * Output workspace should be ```source```
    * Second task is s2i-java and it sould run after git-clone is successfully done
        * Java version should be 11
        * Context path sould be ```todo-api```
        * Produced image sould be ```docker-nexus-edu.tn.hr/todo-api:<your_username>```
        * Workspace where the source resides is ```source```

```
apiVersion: tekton.dev/v1
kind: Pipeline
metadata:
  name: todo-api
  namespace: user1
spec:
  params:
    - default: main
      name: revision
      type: string
  tasks:
    - name: git-clone
      params:
        - name: url
          value: 'https://github.com/true-north-engineering/pbz-educ-src.git'
        - name: revision
          value: $(params.revision)
        - name: refspec
          value: ''
        - name: submodules
          value: 'false'
        - name: depth
          value: '1'
        - name: sslVerify
          value: 'true'
        - name: crtFileName
          value: ca-bundle.crt
        - name: subdirectory
          value: ''
        - name: sparseCheckoutDirectories
          value: ''
        - name: deleteExisting
          value: 'true'
        - name: httpProxy
          value: ''
        - name: httpsProxy
          value: ''
        - name: noProxy
          value: ''
        - name: verbose
          value: 'true'
        - name: gitInitImage
          value: 'registry.redhat.io/openshift-pipelines/pipelines-git-init-rhel8@sha256:dd5c8d08d52e304a542921634ebe6b5ff3d63c5f68f6d644e88417859b173ec8'
        - name: userHome
          value: /home/git
      taskRef:
        kind: ClusterTask
        name: git-clone
      workspaces:
        - name: output
          workspace: source
    - name: s2i-java
      params:
        - name: VERSION
          value: '11'
        - name: PATH_CONTEXT
          value: todo-api
        - name: TLSVERIFY
          value: 'true'
        - name: IMAGE
          value: 'docker-nexus-edu.tn.hr/todo-api:user1'
        - name: BUILDER_IMAGE
          value: 'registry.redhat.io/rhel8/buildah@sha256:5c7cd7c9a3d49e8905fc98693f6da605aeafae36bde5622dc78e12f31db3cd59'
        - name: SKIP_PUSH
          value: 'false'
        - name: ENV_VARS
          value: []
      runAfter:
        - git-clone
      taskRef:
        kind: ClusterTask
        name: s2i-java
      workspaces:
        - name: source
          workspace: source
  workspaces:
    - name: source
```

4. Create todo-frontend pipeline which has the following properties:
    * Define one parameter ```revision``` which will hold the git branch or commit sha of the code to clone.
    * Define one workspace ```source``` which represents the workspace (persistent volume) where the source code will be cloned locally.
    * First task is git-clone:
        * Github URL to clone is ```https://github.com/true-north-engineering/pbz-educ-src.git```
        * Revision to checkout is taken from the ```revision``` parameter. Hint: ```$(params.revision)```
        * Output workspace should be ```source```
    * Second task is s2i-nodejs and it sould run after git-clone is successfully done
        * NodeJS version should be 16-ubi8
        * Context path sould be ```todo-frontend```
        * Produced image sould be ```docker-nexus-edu.tn.hr/todo-frontend:<your_username>```
        * Workspace where the source resides is ```source```

```
apiVersion: tekton.dev/v1
kind: Pipeline
metadata:
  name: todo-frontend
  namespace: user1
spec:
  params:
    - default: main
      name: revision
      type: string
  tasks:
    - name: git-clone
      params:
        - name: url
          value: 'https://github.com/true-north-engineering/pbz-educ-src.git'
        - name: revision
          value: $(params.revision)
        - name: refspec
          value: ''
        - name: submodules
          value: 'false'
        - name: depth
          value: '1'
        - name: sslVerify
          value: 'true'
        - name: crtFileName
          value: ca-bundle.crt
        - name: subdirectory
          value: ''
        - name: sparseCheckoutDirectories
          value: ''
        - name: deleteExisting
          value: 'true'
        - name: httpProxy
          value: ''
        - name: httpsProxy
          value: ''
        - name: noProxy
          value: ''
        - name: verbose
          value: 'true'
        - name: gitInitImage
          value: 'registry.redhat.io/openshift-pipelines/pipelines-git-init-rhel8@sha256:dd5c8d08d52e304a542921634ebe6b5ff3d63c5f68f6d644e88417859b173ec8'
        - name: userHome
          value: /home/git
      taskRef:
        kind: ClusterTask
        name: git-clone
      workspaces:
        - name: output
          workspace: source
    - name: s2i-nodejs
      params:
        - name: VERSION
          value: 16-ubi8
        - name: PATH_CONTEXT
          value: todo-frontend
        - name: TLSVERIFY
          value: 'true'
        - name: IMAGE
          value: 'docker-nexus-edu.tn.hr/todo-frontend:user1'
        - name: BUILDER_IMAGE
          value: 'registry.redhat.io/rhel8/buildah@sha256:5c7cd7c9a3d49e8905fc98693f6da605aeafae36bde5622dc78e12f31db3cd59'
        - name: SKIP_PUSH
          value: 'false'
        - name: ENV_VARS
          value: []
      runAfter:
        - git-clone
      taskRef:
        kind: ClusterTask
        name: s2i-nodejs
      workspaces:
        - name: source
          workspace: source
  workspaces:
    - name: source
```

5. Build ```todo-api``` and ```todo-frontend``` applications using Tekton pipelines. Use the VolumeClaimTemplate for persistence.

6. Check the images in Nexus.

## Task 3 - Deploy the solution

1. Clone the ```https://github.com/true-north-engineering/pbz-educ-src``` Git repo locally and checkout branch with your username. Check the ```helm/todo``` folder and get yourself familliar with the application Helm chart.

2. Clone the ```https://github.com/true-north-engineering/pbz-educ-src``` and ***checkout branch with your username***
    * Change the following properties in ```values.yaml```, commit them and push to the Github repo
        * fe.image.tag should contain the tag of your frontend image
        * api.image.tag should contain the tag of your api image
        * route.host should be ```todo-<your_username>.ocp.pbz.tn.hr```

```
api:
  replicaCount: 1
  image:
    repository: docker-nexus-edu.tn.hr/todo-api
    pullPolicy: IfNotPresent
    tag: "user1"

  imagePullSecrets:
    - name: nexus-docker-auth

  service:
    type: ClusterIP
    port: 8080

  route_path: /todo/api

  livenessProbe:
    tcpSocket:
      port: 8080
  readinessProbe:
    tcpSocket:
      port: 8080

fe:
  replicaCount: 1
  image:
    repository: docker-nexus-edu.tn.hr/todo-frontend
    pullPolicy: IfNotPresent
    tag: "user1"

  imagePullSecrets:
    - name: nexus-docker-auth

  service:
    type: ClusterIP
    port: 3000

  route_path: /

  livenessProbe:
    httpGet:
      path: /
      port: 3000
  readinessProbe:
    httpGet:
      path: /
      port: 3000

route:
  enabled: true
  host: todo-user1.ocp.pbz.tn.hr

mysql:
  auth:
    database: todo
    username: todo
    password: todopassw0rd
    rootPassword: r00tpassw0rd
  networkPolicy:
    enabled: false
  primary:
    pdb:
      create: false
  serviceAccount:
    create: false
```


3. In Argocd ```https://gitops.ocp.pbz.tn.hr``` create a new application with following properties:
    * Application name should be ```<your_username>-todo```
    * Application project should be ```default```
    * Repository URL should be ```https://github.com/true-north-engineering/pbz-educ-src.git```
    * Repository revision should be ```main```
    * Repository path should be ```helm/todo```
    * Deployment target should be local Openshift and namespace that you have creates in Task 2, Step 1
    * Select values files ```values.yaml```

4. Observe the applications in Openshift. Check the application logs and test the application.

```Open the following URL in browser - https://todo-user1.ocp.pbz.tn.hr```

5. Connect to mysql pod and create the necessary tables:

```
mysql -h localhost -utodo -ptodopassw0rd todo

create table Item (id bigint not null auto_increment, description varchar(255), done bool, primary key (id));
CREATE TABLE hibernate_sequence ( next_val BIGINT );
insert into hibernate_sequence values (1);
```

6. Restart the applications and test it.
