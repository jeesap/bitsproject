
pipeline {
     agent any
     
     environment {
     imagename = "jeesap/bitsproject"
     registryCredential = 'jeesap-dockerhub'
     dockerImage = ''
     }
     
     
     stages {
         
         stage('GIT Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/jeesap/bitsproject.git']]])    
            }
        }
        
        
         
         stage('Lint HTML') {
              steps {
                  sh 'tidy -q -e index.html'
              }
         }
         
     
            		stage("Lint Dockerfile") {
			steps {
      		              sh 'sudo /usr/bin/hadolint Dockerfile'
			}
		      	}  
        
        stage('Building image') {
            steps{
             script {
               dockerImage = docker.build imagename
                    }
                 }
            }
        
        stage('Upload Image To Repository') {
            steps{
              script {
               docker.withRegistry( '', registryCredential ) {
               dockerImage.push("$BUILD_NUMBER")
               dockerImage.push('latest')
            }
        }
    }
}

	     
	          stage('Delete the existing deployment') {
                  steps {
                  withAWS(region:'ap-south-1',credentials:'aws') {
                  sh "aws eks --region ap-south-1 update-kubeconfig --name bits"
		#  sh "/usr/local/bin/kubectl delete svc bits"	
		  sh "/usr/local/bin/kubectl get services"
                # sh "/usr/local/bin/kubectl delete deployment bits"
                  sh "/usr/local/bin/kubectl get deployment"
                  sh "/usr/local/bin/kubectl get pod -o wide"
                  }
              }
         }
	     
	     

          stage('Deploy to AWS Kubernetes Cluster') {
                  steps {
                  withAWS(region:'ap-south-1',credentials:'aws') {
                  sh "aws eks --region ap-south-1 update-kubeconfig --name bits"
                  sh "/usr/local/bin/kubectl apply -f deployment.yaml"
                  sh "/usr/local/bin/kubectl get nodes"
                  sh "/usr/local/bin/kubectl get deployment"
                  sh "/usr/local/bin/kubectl get pod -o wide"
                  sh "/usr/local/bin/kubectl apply -f services.yaml"
                  sh "/usr/local/bin/kubectl get services"
                    
                  }
              }
         }


    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $imagename:$BUILD_NUMBER"
         sh "docker rmi $imagename:latest"

      }
    }
 
 
 
     }
}
