https://574521704555.signin.aws.amazon.com/console

Cloudshell Locations-
ak- us-east-1
Arun - us-east-1
Cinjith - us-east-1
Hashif   ap-southeast-2


Nitin - us-east-1 
Manish-us-east-1
Ajeeshregi - us-east-1
AjeeshRavunniyarth - us-east-1


username - sandeep
password - Allianz@2026

https://kubernetes.io/docs/home/

Milan Raj - lab given to Manish Kumar
Chavan Shubham - lab given to Nitin Shrivastava
Muhammed Hashif - lab assigned   -  Sorry joined bit late , can you provide the lab  ?

#aws sts get-caller-identity

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

aws --version (to check aws version)
aws configure
pod
#aws configure
AWS Access Key ID [****************FA6K]: AKIAYLRKPEP
AWS Secret Access Key [****************03aF]: ju8Nfc13xfP/
Default region name [None]: us-east-1
Default output format [None]

#aws sts get-caller-identity
{
    "UserId": "AIDACFBW",
    "Account": "574521704555",
    "Arn": "arn:aws:iam::574521704555:user/ak"
}

Demo- Eks Cluster deployment on aws console with access entry and Node Group config

Cluster config command-
aws eks update-kubeconfig --region us-east-1 --name eks-lab


Conifguring creating configure file for 



aws eks update-kubeconfig --region us-east-1 --name eks-lab
Added new context arn:aws:eks:us-east-1:574521704555:cluster/eks-lab to C:\Users\admin\.kube\config

https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

kubectl get nodes
kubectl get pods -A
kubectl get namespace
kubectl api-resources
kubectl api-resources --namespaced=true.
kubectl config set-context --current --namespace=<namespace_name>   (to switch the namespace)

NOTE: On k8s cluster creation 4 namespace are created:
    1- default - starting ns to start use k8s cluster without any ns creation
    2- kube-system - needs authetication, and it contains resources only for runing my k8s system
    3- kube-node-lease - node specific info, info to send kind of probe to check if node is healthy or failed
    4-kube-public - readable by all, this is like to share public info


kubectl run ak-pod --image=nginx  (to create pod)

kubectl get pod -n <namepspce>
Kubectl describe pod <podname>
kubectl describe pod <podname> -n <n
kubectl exec -it <podname> -- bash
kubectl explain <resource_name>

https://hub.docker.com/

POD:
    
#vim pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: yaml-pod
  namespace: ak
spec:
  containers:
  - name: c1
    image: caddy
  - name: c2
    image: redis
    
#kubectl create -f pod.yaml
#kubectl exec -it yaml-pod -- bash
#kubectl exec -it yaml-pod -- sh
#kubectl exec -it yaml-pod -c c2 -- sh

LABELS:
    kubectl label pod pod2 env=qa user=allianz
    kubectl get pod --show-labels
    kubectl label pod pod2 env=prod user=newuser --overwrite
   kubectl get pod --show-labels
 kubectl label pod pod2 user-
 
 label through yaml:
apiVersion: v1
kind: Pod
metadata:
  name: yaml-pod
  namespace: ak
  labels:
    env: dev
    user: ak
spec:
  containers:
  - name: c1
    image: caddy
  - name: c2
    image: redis

REPLICA SET:
#vi rs.yaml

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-ak
  namespace: ak
spec:
  replicas: 5
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: c1
        image: nginx
      - name: c2
        image: redis
        
    #kubectl create -f rs.yaml
    #kubectl get pods
    #kubectl get rs
    
     kubectl delete rs rs-ajeeshregi -n <namespace_name>  (to delete from other namespace)
    
DEPLOYMENT: Alternative to deployment is STATEFULSET(to explore)
    #vi dep.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep-ak
  namespace: <namespace>
  annotations:
    kubernetes.io/change-cause: "first revision"
spec:
  replicas: 5
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: c1
        image: nginx
      - name: c2
        image: redis:8.6-trixie
    
    #kubectl create -f dep.yaml
    After creating now try modifying image tags or change image name and also update annotations then apply it
    after 3 -4 modificatrion now let us see the history
    #kubectl rollout history deploy dep-ak
    #kubectl rollout undo deploy dep-ak
    #kubectl rollout undo deploy dep-ak --to-revision=1
    #kubectl get rs (to see how deployment manages all replic sets for you)
    #kubectl get deploy dep-ak -o yaml  (to check for default values of deployment)
    #kubectl get pods -w
    
    SERVICES:
    ClusterIP:
        #vi cip.yaml
    
apiVersion: v1
kind: Service
metadata:
  name: cipsvc-ak
  namespace: namespace
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80  #service port inside cluster
    targetPort: 80 #container port
    
    #vi dep.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep-cip
  namespace: namespace
  annotations:
    kubernetes.io/change-cause: "image tag updating to caddynline-perl for nginx"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: c1
        image: caddy
    #kubectl get svc
    Note cluster Ip
    
    To be in same network lets access worker node shell:
        #kubectl debug node/<nodename> --image ubuntu  (this opens shell access of the defined node)
        kubectl debug node/ip-10-0-140-249.ec2.internal -it --image=ubuntu -- bash ---- third party terminal 
        insode the node install curl(apt-get update -y, apt-get install curl -y)
        #curl <svc-ip>:80  (this should give access to application
        
    NodePort:
#vi np.yaml
apiVersion: v1
kind: Service
metadata:
  name: npsvc-ak
  namespace: namespacename
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80  #service port inside cluster
    targetPort: 80 #container port
    nodePort: 30020
    
    #vi dep.ayml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep-np
  namespace: namespace
  annotations:
    kubernetes.io/change-cause: "image tag updating to caddynline-perl for nginx"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: c1
        image: caddy
        
    #kubectl get svc
    Now fetch node ip
    #kubect get node -o wide
    copy any node ip
    
    now again as we dont have public ip of worker nodes enabled in eks let us check using internal ip of node, so access shell of node again as above:
        #kubectl debug node/<nodename> --image ubuntu  (this opens shell access of the defined node
        insode the node install curl(apt-get update -y, apt-get install curl -y)
        #curl <node-ip>:<node-port>  (this should give access to application  
        
    LoadBalancer:
    #vi lb.yaml
    
apiVersion: v1
kind: Service
metadata:
  name: lbsvc-ak
  namespace: namespacename
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80  #service port inside cluster
    targetPort: 80 #container port
    #nodePort: 30020
  
   #kubectl create -f lb.yaml

    #kubectl get svc (notice externla lb dns name)
    
    #vi dep.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep-lb
  namespace: namespace
  annotations:
    kubernetes.io/change-cause: "image tag updating to caddynline-perl for nginx"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: c1
        image: caddy

#kubectl describe svc <svcname> (confirm the endpoints)

Open new tab in browser and check with external ip dns name of svc to access the application

Deployment Strategies:
1-RollingUpdate - replace old pods with new pods and no downtime
2-Recreate - all exisiting pods are terminated and then new ones are created so there will be some downtime
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep-new
  namespace: ak
  annotations:
    kubernetes.io/change-cause: "image tag updating to caddynline-perl for nginx"
spec:
  strategy:
    type: Recreate
  replicas: 3
  selector:
    matchLabels:
      app: caddy
  template:
    metadata:
      labels:
        app: caddy
    spec:
      containers:
      - name: c1
        image: caddy

Blue-Green Deployment
Two deployments running at same time but service pointint to any one only,
once new deployment is tested we change the selector label in service to point to new deployment

apiVersion: v1
kind: Service
metadata:
  name: lbsvc-ak
  namespace: ak
spec:
  type: LoadBalancer
  selector:
    app: caddy (update this to point to label selector of new deployment to switch)
  ports:
  - port: 80  #service port inside cluster
    targetPort: 80 #container port
    #nodePort: 30020

Canary deployment: 
dep-v1: nginx(80%)
label: app: web


#cat nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-v1
  namespace: ak
  annotations:
    kubernetes.io/change-cause: "image tag updating to caddynline-perl for nginx"
spec:
  replicas: 4
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: c1
        image: nginx

dep-v2: caddy(20%)
label: app: web

#cat caddy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-v2
  namespace: ak
  annotations:
    kubernetes.io/change-cause: "image tag updating to caddynline-perl for nginx"
spec:
  replicas: 4
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: c1
        image: caddy

svc the label will match app: web


#cat lb.yaml
apiVersion: v1
kind: Service
metadata:
  name: canary-lb
  namespace: ak
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80  #service port inside cluster
    targetPort: 80 #container port
    #nodePort: 30020

Environment variables:

It can be defined using two feilds:
env - set variable for container directly  (plain/text format)
envfrom - refer a configmap/secret

#cat envpo.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysqlpod
  namespace: ak
  labels:
    env: dev
    user: ak
spec:
  containers:
  - name: c1
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: root123
    - name: MYSQL_DATABASE
      value: allianzdb

kubectl create -f envpo.yaml
#kubectl exec -it mysqlpod -- bash
(inside pod) #env (to check variable
#mysql -u root -p (when prompted enter password)
(inside mysql) #show databases;  (to check db created)

Configmap: api object which is used to store data(non confidential data) in key value pairs
#cat cfmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm1
  namespace: ak
data:
  MYSQL_ROOT_PASSWORD: "root@2026"
  MYSQL_USER: "allianz"
  MYSQL_PASSWORD: "allianz123"
  MYSQL_DATABASE: "newdb"

create a pod to consume this configmap:
#cat cmpod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysqlpod-new
  namespace: ak
  labels:
    env: dev
    user: ak
spec:
  containers:
  - name: c1
    image: mysql
    envFrom:
    - configMapRef:
        name: cm1

Now if we want to call only certain keys from the configmap then:
#cat cfmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm1
  namespace: ak
data:
  root-pass: "root@2026"
  MYSQL_USER: "allianz"
  MYSQL_PASSWORD: "allianz123"
  sql-db: "newdb"

and then pod to call this:
#cat newpod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysqlpod-new
  namespace: ak
  labels:
    env: dev
    user: ak
spec:
  containers:
  - name: c1
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
        configMapKeyRef:
          name: cm1
          key: root-pass
    - name: MYSQL_DATABASE
      valueFrom:
        configMapKeyRef:
          name: cm1
          key: sql-db


Secret: store sensitive data such as password, token or keys, encoded as base64
$ cat secret.yaml 
apiVersion: v1
kind: Secret
metadata:
  name: sec1
  namespace: ak
stringData:
  MYSQL_ROOT_PASSWORD: "allianz123"
  MYSQL_USER: "user1"
  MYSQL_PASSWORD: "user123"
 
$ cat secretpod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: secpod
  namespace: ak
spec:
  containers:
  - name: c1
    image: mysql
    envFrom:
    - secretRef:
        name: sec1


RBAC - users, groups
K8s does not have user object to manage

ServiceAccount - application identity(non-human account)

#kubectl auth can-i --as=system:serviceaccount:<namespace-name>:<sa-name> -n <namespace-to-check>
#kubectl auth can-i --as=system:serviceaccount:ak:default get pod -n ak
#kubectl auth can-i --list --as=system:serviceaccount:ak:default -n ak

IAM USER -->token-->api server


Create Service account:
#kubectl create sa <sa_name>

check new sa created does not have even read permission on the pod
#kubectl auth can-i --as=system:serviceaccount:<namespace-name>:<sa-name> -n <namespace-to-check> get pod

to modify:
Create role
#cat role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-read
  namespace: ak
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get","list"]

#cat rolebind.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: podreadbind
  namespace: ak
subjects:
  - kind: ServiceAccount
    name: aksa
    namespace: ak
roleRef:
  kind: Role
  name: pod-read
  apiGroup: rbac.authorization.k8s.io

 now again check
#kubectl auth can-i --as=system:serviceaccount:<namespace-name>:<sa-name> -n <namespace-to-check>  (should give yes)

HELM: A package manager for kubernetes
https://helm.sh/docs/intro/install/ 
https://artifacthub.io/

helm version
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh 
helm version 

helm repo add allianz https://charts.bitnami.com/bitnami ---create repo from public repository
helm repo add ak-repo url/compnay.com/charts --username ak --password akpass  -- to add reop from private helm repository
helm repo list --- show list of repos
helm repo update allianz --- update/pull the packages
helm search repo allianz  --- show repo pakages
helm show chart allianz/nginx --- show chart of nginx
helm show values allianz/nginx --- show values
helm install nginxak allianz/nginx --- install chart wiht name from repo added
kubectl get pod
kubectl get deploy
kubectl get svc
helm show values allianz/nginx | grep replica --- For windows cmd run this helm show values allianz/nginx | findstr replica
helm pull allianz/nginx --untar
ls
cd nginx/
ls -- alternate for cmd dir /P
cat Chart.yaml --- alternate for cmd type Chart.yaml
cat values.yaml ---
 vi values.yaml ---alternate notepad values.yaml
cd ..
helm upgrade nginxak ./nginx -f ./nginx/values.yaml - to rollout new update after updating values
kubectl get pod
kubectl get deploy
helm uninstall nginxak - delete chart and deployment

Create chart:
#helm create chartname (this will create folder with chartname that contains Chart.yaml, values.yaml, templates/ and charts)

helm create mywebapp
helm repo list (you wont see chart this is repolist)
ls
ls mywebapp/
ls mywebapp/templates/
cd mywebapp/
cat Chart.yaml 
vi Chart.yaml --- update the version from 0.1.0 to 0.1.1
vi values.yaml --- update the replica from 1  to 2
ls templates/
vi templates/deployment.yaml 
vi templates/service.yaml 
ls templates/tests/
cd ..
helm install myapp mywebapp
kubectl get pods
kubectl get svc
kubectl get svc -A
helm template test mywebapp
vi mywebapp/values.yaml (modify replicacount)
vi mywebapp/Chart.yaml (update chart version)
helm upgrade myapp mywebapp
helm history myapp
helm rollback myapp 1
kubectl get pod
helm history myapp
+


curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp



GIT ---
if not installed on local cli then install git bash https://git-scm.com/install/windows

Usecase:
1- version history
2- rollback
3- team conflicts

Advantages:
1- Version control
2- Collaboration
3- History tracking
4- Rollback

Repository:
This is your storage of project files:
Source code
Configuration
History
Branches


Git config mandatory:
user.name
user.email

Commands:
          8  mkdir gitdir
    9  cd gitdir
   10  git init
   11  ls -a
   12  git config --list
   13  git config --global user.name "ak"
   14  git config --global user.email "ak@gmail.com"
   15  git config --list
   16  git status
   17  vi app.txt
   18  git status
   19  git add app.txt 
   20  git status
   21  git commit -m "initial first commit on new file created"
   22  git status
   23  vi app.txt 
   24  git status
   25  git add app.txt 
   26  git commit -m "second commit after file modification"
   27  git status
   28  vi app.txt 
   29  git status
   30  git diff
   31  git add .
   32  git commit -m "third commit after third change of line"
   33  git status
   34  ls
   35  git log --oneline

To push this on remote github repo:
   37  git branch
   38  git branch -M main
   39  git branch
   40  git remote add allianz https://github.com/ak9675/allainztest.git
   41  git remote -v
   42  git status
Now set up the settings in the user profile and generate token for authentication
   43  git push -u allianz main (enter username and token as password when prompted)

Branch:
    11 git branch
   12  git branch feature-f1
   13  git branch
   14  git checkout feature-f1
   15  git branch
   16  cat app.txt 
   17  echo "this is new line for first branch feature-f1" >> app.txt 
   18 
   19  git status
   20  git add .
   21  git commit -m "new branch new line added to thefile"
   22  git status
   23  git log --oneline
   24  cat app.txt 
   25  git checkout main
   26  cat app.txt 
   27  git checkout feature-f1
   28  cat app.txt 
   29  git status
   30  git push -u allianz feature-f1

Trunk- based:
From main branch --- create short lived branches and deploy the change to main and then delete it, so that long lived branches does not create complications in future

CI-CD

Git stores code --> Validate code CI ---> Deliver/deply the code CD-->GitOps deploy the code


Developer  --->compiling-->testing-->building-->deploying (maunal tasks)
error in manual approach
slow delivery due to time taken manually
inconsistency

To remove these we have Continous Integration (CI)
This means automatically build and test code whenever a new change is commited to github repo

Git push--> GHA(github action)-->Building docker image-->run tests (CI will ends once code is build,tested and validated)

Continuous Delivery (CD) -  deploy code but requires manual approval

git push-->build--> test-->validate-->artifacts--.deployed(approval stage)-->ready for production

Continuos Deployment(CD) -- every succesful change automatically is deployed without manual approval fully automatic

Demo:

Current flow--
Local/cloudshell folder/files--->pusing to github with commit and branch-->stored on github

New flow--
Local/cloudshell folder/files--->pusing to github with commit and branch-->stored on github with new change-->Github Actions-->build and test the code stored


Setting up first CI pipeline:
mkdir .github
mkdir .github/workflows
ls -a
vi .github/workflows/ci.yaml
#cat .github/workflows/ci.yaml 
name: first CI pipeline

on:
  push:
    branches:
      - main

jobs:
  build:

    runs-on: ubuntu-latest

    steps:

    - name: Checkout Code
      uses: actions/checkout@v4

    - name: Print message
      run: echo "Hello world from github actions"

    - name: show files
      run: ls -al

git status
git add .
git status
git commit -m "ci pipeline file added"
git push -f allianz main

Now in github note in github actions an action should be created and deployed



PARKING: RBAC demo

Ingress
HELM
Install eksctl:
#curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
#sudo mv /tmp/eksctl /usr/local/bin
#eksctl version


