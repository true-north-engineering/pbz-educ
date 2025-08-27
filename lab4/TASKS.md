# Lab 1 - CI, CD and GitOps

Before starting with lab exercises connect to the linux system box-edu.tn.hr with ssh or putty as userX where X is a number designated to you by the presenter.
Example:

```ssh userX@box-edu.tn.hr```

## Task 1 - S2I build

1. Clone the pbz-educ-src in your home folder.

```git clone https://github.com/true-north-engineering/pbz-educ-src.git```

2. Generate Containerfile for source in ```pbz-educ-src/todo-frontend``` based on the builder image ```registry.access.redhat.com/ubi8/nodejs-16:latest```

3. Build the container image from the generated Containerfile

## Task 2 - Tekton builds

1. Create an Openshift project called by your username, e.g. user1

2. Create ```nexus-docker-auth``` secret that containes authentication data for ```docker-nexus-edu.tn.hr```. Link that secret with ```pipeline``` service account.

3. Connect to the Openshift console. `pbz-cicd` project contains Tekton pipeline examples. Copy them to your project.

4. Create todo-api pipeline which has the following properties:
    * Define one parameter ```revision``` which will hold the git branch or commit sha of the code to clone.
    * Define one workspace ```source``` which represents the workspace (persistent volume) where the source code will be cloned locally.
    * First task is git-clone:
        * Github URL to clone is ```https://github.com/true-north-engineering/pbz-educ-src.git```
        * Revision to checkout is taken from the ```revision``` parameter. Hint: ```$(params.revision)```
        * Output workspace should be ```source```
    * Second task is s2i-java-1-19-0 and it sould run after git-clone is successfully done
        * Java version should be 11
        * Context path sould be ```todo-api```
        * Produced image sould be ```docker-nexus-edu.tn.hr/<your_username/todo-api:latest```
        * Workspace where the source resides is ```source```

5. Create todo-frontend pipeline which has the following properties:
    * Define one parameter ```revision``` which will hold the git branch or commit sha of the code to clone.
    * Define one workspace ```source``` which represents the workspace (persistent volume) where the source code will be cloned locally.
    * First task is git-clone:
        * Github URL to clone is ```https://github.com/true-north-engineering/pbz-educ-src.git```
        * Revision to checkout is taken from the ```revision``` parameter. Hint: ```$(params.revision)```
        * Output workspace should be ```source```
    * Second task is s2i-nodejs-1-19-0 and it sould run after git-clone is successfully done
        * NodeJS version should be 18-ubi8
        * Context path sould be ```todo-frontend```
        * Produced image sould be ```docker-nexus-edu.tn.hr/<your_username/todo-frontend:latest```
        * Workspace where the source resides is ```source```

6. Build ```todo-api``` and ```todo-frontend``` applications using Tekton pipelines. Use the VolumeClaimTemplate for persistence.

7. Check the images in Nexus.

## Task 3 - Deploy the solution

1. Clone the ```https://github.com/true-north-engineering/pbz-educ-src``` Git repo locally (or on box-edu.tn.hr) and create branch with your username. Check the ```helm/todo``` folder and get yourself familliar with the application Helm chart.

Hint to create the branch with git cli:

```
git clone https://github.com/true-north-engineering/pbz-educ-src.git
cd pbz-educ-src
git config user.email=user@educ.tn.hr
git config user.name=user

git checkout -b <your_username>

#make changes to files

git add .
git commit -m "comment"
git push

# When asked for username enter "user". Password can be found in file /edu/github-token
```

2. In the cloned ```https://github.com/true-north-engineering/pbz-educ-src```
    * Change the following properties in ```values.yaml```, commit them and push to the Github repo
        * fe.image.tag should contain the tag of your frontend image
        * api.image.tag should contain the tag of your api image
        * route.host should be ```todo-<your_username>.ocp-edu.tn.hr```

3. In Argocd ```https://gitops-edu.tn.hr``` create a new application with following properties:
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
