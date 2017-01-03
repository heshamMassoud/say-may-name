# say-my-name
A spring-boot microservice to demonstrate docker container orchestration using a hosted kubernetes cluster on Google Cloud's Container Engine.

###About the sample microservice

###What the Prototype Covers
1. A deployed instance of Jenkins server on a Kubernetes cluster.
2. Setup of a multi-branched pipeline on Jenkins.
3. Setup of the a Production and Staging environment for the microservice on the kubernetes cluster.
4. Development workflow and release to production (and how to switch it to canary-releasing).
5. Manual horizontal scaling.
6. Auto-scaling.
7. Monitoring and logging using Stackdriver.
8. Dynamic Configuration.
9. Advantages of using Kubernetes on GKE.

###Bird's Eye View

####Environment Isolation
Currently the staging environment is totally isolated from the production environment using kubernetes namespaces to
have pod-level isolation
```bash
kubectl --namespace=staging apply -f k8s/services/staging
kubectl --namespace=staging apply -f k8s/deployments/staging
```
The services know which pods to connect to using labels and selectors as follows:
######The Pod
```bash
labels:
    app: say-my-name
    role: frontend
    env: staging
```
######The Service
```bash
selector:
    app: say-my-name
    role: frontend
    env: staging
```
####Canary-Releases
However, it is possible to have the staging environment set as a canary release environment, in which case 
both environments would have the same namespace and only updating the staging container images. This would lead, for example,
 to having a new update seen by 20% of the users and if it's okay, it can be rolled out to the production pods.
####Jenkins Multi-branched Pipeline
<img width="880" alt="screen shot 2017-01-03 at 19 12 56" src="https://cloud.githubusercontent.com/assets/9512131/21618084/80643c54-d1e9-11e6-9c28-265f714081bc.png">
#####Manually setup a kubernetes .yaml deployment
```bash
kubectl --namespace=staging apply -f k8s/deployments/staging/
```

#####Manual horizontal scaling of production environment pods
```bash
kubectl --namespace=production scale deployment say-my-name-frontend-production --replicas=4
```

#####Status of created services and pods
```bash
kubectl --namespace="staging" get services
kubectl --namespace="staging" get pods
```

#####Details of created services and pods
```bash
kubectl --namespace="staging" describe services
kubectl --namespace="staging" describe pods
```

##Stack driver
###Monitoring
![screen shot 2017-01-03 at 21 06 48](https://cloud.githubusercontent.com/assets/9512131/21621291/a18d7cba-d1f8-11e6-9a02-8849eaad9160.png)

###Error Reporting and Logging
To make utilize the full advantages of stack driver's error reporting and logging, one has to use one of the client libraries provided by google which are still in beta and not recommended for production purposes.
https://cloud.google.com/error-reporting/docs/setup/compute-engine#log_exceptions

