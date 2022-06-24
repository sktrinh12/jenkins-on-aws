# Jenkins

## Summary

Jenkins is a CI/CD tool to automate DevOps tasks/jobs. It can be extrapolated to do other automated processes like ETLs. It has been used to allow developers to write code and push to GitHub which would trigger a Jenkins job to build and deploy to production.

Jenkins works by spinning up a jenkins agent (slave) that has a particular set of permissions baked into it. It works like a server-less entity whereafter it completes the job/task at hand, it will spin down and free up resources. 

The github repo contains all the `.yaml` files that were used to deploy the jenkins application onto a running kubernetes cluster. See 

There also also pipelines scripts that instruct the Jenkins agent to carry out jobs/tasks. The `Dockerfile` is based off of: [https://hub.docker.com/r/jenkins/slave/dockerfile/](https://hub.docker.com/r/jenkins/slave/dockerfile/). This `Dockerfile` is the jenkins agent that is responsible for running the job. 

## Prequisites

- kubectl
- running kubernetes cluster
- shared drive; in this case a EFS was procured on AWS [https://aws.amazon.com/efs/](https://aws.amazon.com/efs/) [https://aws.amazon.com/blogs/storage/deploying-jenkins-on-amazon-eks-with-amazon-efs/](https://aws.amazon.com/blogs/storage/deploying-jenkins-on-amazon-eks-with-amazon-efs/)

## Instructions

1. Deploy the `jenkins-pv.yaml` which is the persistent volume
2. Deploy the `jenkins-pvc.yaml` which is the persistent volume claim
3. Deploy the `jenkins-rbac.yaml` which is the role based access control manifest
4. Deploy the `jenkins-deploy.yaml` which is the main deployment manifest
5. Deploy the `jenkins-svc.yaml` which will create the service to access the pods

```bash
kubectl apply -f jenkins-pv.yaml
kubectl apply -f jenkins-pvc.yaml
kubectl apply -f jenkins-rbac.yaml
kubectl apply -f jenkins-deploy.yaml
kubectl apply -f jenkins-svc.yaml
```

Once the Jenkins cluster is up and running, run a `port-forward` to access it on local computer.

```bash
kubectl port-forward -n jenkins {POD_NAME} 8080:8080
```

Go into the UI and install the plugin, `kubernetes-plugin` by clicking on ‘Manage Plugins’

- Go to Manage Jenkins | Bottom of Page | Cloud | Kubernetes (Add kubenretes cloud)
- Fill out plugin values
    - Name: kubernetes
    - Kubernetes URL: [https://kubernetes.default](https://kubernetes.default/)
    - Kubernetes Namespace: jenkins
    - Credentials | Add | Jenkins (Choose Kubernetes service account option & Global + Save)
    - Test Connection | Should be successful! If not, check RBAC permissions and fix it!
    - Jenkins URL: [http://jenkins](http://jenkins/)
    - Tunnel : jenkins:50000
    - Apply cap only on alive pods : yes!
    - Add Kubernetes Pod Template
        - Name: jenkins-slave
        - Namespace: jenkins
        - Service Account: jenkins
        - Labels: jenkins-slave (you will need to use this label on all jobs)
        - Containers | Add Template
            - Name: jnlp
            - Docker Image: sktrinh12/jenkins-slave
            - Command to run :
            - Arguments to pass to the command:
            - Allocate pseudo-TTY: yes
            - Add Volume
                - HostPath type
                - HostPath: /var/run/docker.sock
                - Mount Path: /var/run/docker.sock
        - Timeout in seconds for Jenkins connection: 300
- Save

---

The 'Docker Image' field would be the tag name of the Docker image that was created from the `Dockerfile`.
