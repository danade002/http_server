pipeline {

    parameters {
        string(name: 'ENVIRONMENT', defaultValue: 'production', description: 'Deployment environment')
        string(name: 'BRANCH', defaultValue: 'production', description: 'Branch of application to deploy')
    }
 
    

    agent { label 'master' }

    stages {
        stage('clone source repo') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[ name: "*/$BRANCH" ]],
                    userRemoteConfigs: [[ 
                        credentialsId: 'adjust-test',
                        url: 'git@github.com:danade002/http_server.git'
                    ]]
                    
                   
                    
                
                ])
           
            }
        }



        stage('Build and Push Image') {
            
            
            steps {
                        withCredentials([usernamePassword(credentialsId: 'dockerhubLogin', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]){
                _sh """
                ls -la
                echo "built image"
                docker build . -t danielademeso/adjusttest:${BUILD_NUMBER} 
                docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}
                
                echo "pushed image"
                docker rmi danielademeso/adjusttest:$BUILD_NUMBER
                echo "removed image"
		docker system prune -f 
		echo "removed possible dangling image"
                """
            }
            }
        }

        stage('Deploy to Dev Kubernetes') {
              environment {
                KUBECONFIG = credentials('KUBECONFIG')
            }
            
            
            steps {
                    _sh """
                  
		    kubectl --kubeconfig=${KUBECONFIG}  --namespace=$ENVIRONMENT set image deployment/http-server http-server=danielademeso/adjusttest:${BUILD_NUMBER}
                    """
                
            }
        }
    }
}

def _sh (shell_command) {
    sh """
    #!/bin/bash -e

    $shell_command
    """
}
