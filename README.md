# JenkinsHelmChart on AWS EC2 - Ubuntu:

Setup Jenkins using Helm Chart on AWS EC2

1. Step Ubuntu EC2 Small Instance of 2 Core vCPUs and 2GiB Memory using default VPC Configuration

2. Install Kubectl with following commands:
    
    Download the latest release with the command:
    
        curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
    
    Change File Permission:
        
        chmod +x ./kubectl
        
    Move Kubectl File to /usr/local/bin Folder:
    
        sudo mv ./kubectl /usr/local/bin/kubectl
        
3. Install Docker:

    Download the latest version of Docker using following command:
    
        sudo apt-get update && sudo apt-get install docker.io -y
        
    Add Local User to Docker Group:
        
        sudo gpasswd -a ubuntu docker
        
    Update Permission of Docker.sock File:
    
        sudo chmod 666 /var/run/docker.sock
 
 4. Install MiniKube on EC2 Instance:
 
    Download the latest version of MiniKube using following command:
    
        curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
        
    Check MiniKube Version:
    
        minikube version
        
    Switch to Root User for Installation:
    
        sudo su
        
    Installed Minikube with --vm-driver=none because we are getting virtualisation on VM:
    
        minikube start --vm-driver=none
      
    Command to Check Installation Status:
    
        minikube status
        
 5. Check Kubernetes Nodes, Pods and Cluster Details:
 
     But before run following to set KUBECONFIG value in environment values else it will give error "The connection to the server localhost:8080 was refused - did you specify the right host or port?"
     
     	sudo cp /etc/kubernetes/admin.conf $HOME/
		sudo chown $(id -u):$(id -g) $HOME/admin.conf
		export KUBECONFIG=$HOME/admin.conf
        
     Followed by following commands:
     
        kubectl get nodes  
	    kubectl get pods --all-namespaces
	    kubectl cluster-info
	    kubectl config get-contexts
        
 6. Install Helm:
 
    Download the latest version of Helm:
    
        cd /tmp/
        curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > install-helm.sh
        
    Change the File Permission and execute install file:
    
        chmod u+x install-helm.sh
        ./install-helm.sh
        
    Create tiller service account:
        
         kubectl -n kube-system create serviceaccount tiller
         
    Create cluster role binding for tiller:
    
        kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
        
    Install socat package:
    
        sudo apt-get -y install socat
        
    Initialize Tiller:
    
        helm init --service-account tiller
        
 8. Install Jenkins Server using Helm Chart:
 
    Search for Jenkins Chart in Helm:
    
        helm search jenkins
        
     Take Copy of value.yaml File:
     
        helm inspect values stable/jenkins > /tmp/jenkins.values
        
     Updated jenkins.values file with following parameters:
     
        Change Resource Configuration:
      
           resources:
              requests:
                cpu: "50m"
                memory: "256Mi"
              limits:
                cpu: "500m"
                memory: "1024Mi
                
        Set ServiceType to NodePort in order to access Jenkins outside cluster:
       
            serviceType: NodePort
            NodePort: 32329
         
        Change the port value from default 8080 t0 8085:
       
            servicePort: 8085
            targetPort: 8085
            
       Install Jenkins using updated jenkins.values file
       
             helm install stable/jenkins --values /tmp/jenkins.values --name myjenkins
             
       Follow the instructions that are printed from NOTES.TXT to access Jenkins 
       
               1. Get your 'admin' user password by running:
                    printf $(kubectl get secret --namespace default myjenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
               2. Get the Jenkins URL to visit by running these commands in the same shell:
                    export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=myjenkins" -o jsonpath="{.items[0].metadata.name}")
                    echo http://127.0.0.1:8080
                    kubectl --namespace default port-forward $POD_NAME 8080:8080
                    
  9. Create JenkinsFile to run the following job in Jenkins:
  
        


     
        
        
        
   
   
        
        
     
        
    
    
      
