# Lab 1 - CI, CD and GitOps

Before starting with lab exercises connect to the linux system box-edu.tn.hr with ssh or putty as userX where X is a number designated to you by the presenter.
Example:

```ssh userX@box-edu.tn.hr```

## Task 1 - S2I build

1. Clone the pbz-educ-src in your home folder.

```git clone https://github.com/true-north-engineering/pbz-educ-src.git```

2. Generate Containerfile for source in ```pbz-educ-src/todo-frontend``` based on the builder image ```registry.access.redhat.com/ubi8/nodejs-16:latest```

3. Build the container image from the generated Containerdile

## Task 2 - Tekton builds

1. Create an Openshift project called by your username, e.g. user1

2. Create ```nexus-docker-auth``` secret that containes authentication data for ```docker-nexus-edu.tn.hr```. Link that secret with ```pipeline``` service account.

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

5. Build ```todo-api``` and ```todo-frontend``` applications using Tekton pipelines. Use the VolumeClaimTemplate for persistence.

6. Check the images in Nexus.

## Task 3 - Deploy the solution

1. Clone the ```https://github.com/true-north-engineering/pbz-educ-src``` Git repo locally and checkout branch with your username. Check the ```helm/todo``` folder and get yourself familliar with the application Helm chart.

2. Clone the ```https://github.com/true-north-engineering/pbz-educ-src``` and ***checkout branch with your username***
    * Change the following properties in ```values.yaml```, commit them and push to the Github repo
        * fe.image.tag should contain the tag of your frontend image
        * api.image.tag should contain the tag of your api image
        * route.host should be ```todo-<your_username>.ocp.pbz.tn.hr```

3. In Argocd ```https://gitops.ocp.pbz.tn.hr``` create a new application with following properties:
    * Application name should be ```<your_username>-todo```
    * Application project should be ```default```
    * Repository URL should be ```https://github.com/true-north-engineering/pbz-educ-src.git```
    * Repository revision (branch) should be your username
    * Repository path should be ```helm/todo```
    * Deployment target should be local Openshift and namespace that you have creates in Task 2, Step 1
    * Select values files ```values.yaml```

4. Observe the applications in Openshift. Check the application logs and test the application.

5. Connect to mysql pod and create the necessary tables:

```
mysql -h localhost -utodo -ptodopassw0rd todo

create table Item (id bigint not null auto_increment, description varchar(255), done bool, primary key (id));
CREATE TABLE hibernate_sequence ( next_val BIGINT );
insert into hibernate_sequence values (1);
```

6. Restart the applications and test it.