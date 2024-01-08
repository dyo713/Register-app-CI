
pipeline {
    agent { label 'Jenkins-Agent' }
    // tools {
    //     jdk 'Java17'
    //     maven 'Maven3'
    // }
    environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "sailesh94"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }	
    
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'token', url: 'https://github.com/kaulaskar/Register-app-CI.git'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }
       // stage("SonarQube Analysis"){
       //     steps {
	      //      script {
		     //    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
       //                  sh "mvn sonar:sonar"
		     //    }
	      //      }	
       //         }
       //      }

       stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('','dockerhub') {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

       }
       // stage("Trivy Scan") {
       //     steps {
       //         script {
	      //       sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ashfaque9x/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
       //         }
       //     }
       // }

       stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       } 
       // stage('connect k8') {
       //      steps {
       //          script {
       //              sh "sudo kubectl get nodes"
                   
       //         }
                
       //      }
       //  }	

       // stage('deploy k8 files') {
       //      steps {
                
       //        sh '''
       //                   sudo kubectl apply -f $WORKSPACE/deploy.yaml
       //        '''
                
       //      }
       //  }    
       // stage('validate deployment') {
       //      steps {
       //          sh '''
       //           sudo kubectl get po
                 
       //          '''
       //      }
       //  }
      stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user sailesh:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-18-206-100-26.compute-1.amazonaws.com:8080/job/jenkins_k8/job/Cd_k8/buildWithParameters?token=gitops-token'"
                }
            }
       }	    
    }

}
