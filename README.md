# angular_app_eks
deploy angular app ci-cd to eks

<img width="1197" alt="image" src="https://user-images.githubusercontent.com/61488445/157611994-1e1bf78f-b859-49c2-bc2b-9309343adc19.png">


<img width="935" alt="image" src="https://user-images.githubusercontent.com/61488445/157612179-37088bb7-4124-4d1f-b48a-f9a4c2b3feaf.png">

```
Jenkins Pipeline script:

node
{
  def mvnHome = tool 'M3'
     stage ('checkout') {
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'GIT_HUB_CREDENTIALS', url: 'https://github.com/saranca14/angular_app_eks.git']]])      
        }
   
    stage ('Build') {
            sh 'mvn -f pom.xml clean install'           
        }
       
       
    stage ('Docker Build') {
         // Build and push image with Jenkins' docker-plugin
            withDockerRegistry([credentialsId: "dockerhub", url: "https://index.docker.io/v1/"]) {
            image = docker.build("saranrnair/mywebapp")
            image.push()    
            }
        }

      stage ('K8S Deploy') {
       
                kubernetesDeploy(
                    configs: 'springboot-lb.yaml',
                    kubeconfigId: 'K8S',
                    enableConfigSubstitution: true
                    )               
        }
    
}
```
Services running in EKS:

```
kubectl get nodes
NAME                                           STATUS   ROLES    AGE     VERSION
ip-172-31-11-19.ap-south-1.compute.internal    Ready    <none>   7m40s   v1.21.5-eks-9017834
ip-172-31-40-195.ap-south-1.compute.internal   Ready    <none>   7m50s   v1.21.5-eks-9017834
```
```
kubectl get deployments

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
my-ak-deployment   2/2     2            2           13m
```
```
kubectl get svc
NAME              TYPE           CLUSTER-IP     EXTERNAL-IP                                                               PORT(S)          AGE
angular-app-svc   LoadBalancer   10.100.85.16   a6ff4a29b604b4729bb1ef3be4433a0f-256914431.ap-south-1.elb.amazonaws.com   8085:30653/TCP   14m
kubernetes        ClusterIP      10.100.0.1     <none>
```

Search in browser with 'a6ff4a29b604b4729bb1ef3be4433a0f-256914431.ap-south-1.elb.amazonaws.com:8085'
