pipeline{
    agent any
    environment{ 

        VERSION = "${env.BUILD_ID}"
    } 

    stages{
        stage("sonar quality status"){

            agent {

                docker {
                    image 'maven'
                }
            }
            steps{
                script{

                    withSonarQubeEnv(credentialsId: 'sonar-token') {

                        sh'mvn clean package sonar:sonar'       
                } 
                    
                }  
            }    
        }

        stage('Quality gate status'){

            steps{

                script{

                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'   
                }
            }
        }
        stage('docker build & docker push to Nexus Repo'){

            steps{

                script{
                    
                    withCredentials([string(credentialsId: 'admin_nexus_passwd', variable: 'nexus_creds')]) {
                    sh ''' 
                     docker build -t 3.84.125.50:8083/springapp:${VERSION} .

                     docker login -u admin -p $nexus_creds 3.84.125.50:8083

                     docker push 3.84.125.50:8083/springapp:${VERSION}

                     docker rmi 3.84.125.50:8083/springapp:${VERSION}
                     
                    '''

                    }
                }
            }
        }
        stage('Identify misconfig using datree in the helm charts'){

            steps{

                script{

                    dir('kubernetes/myapp/') {
                         withEnv(['DATREE_TOKEN=0efa386a-825b-45a1-ab80-46a74870a6c3']) {
                        sh 'helm datree test .'
                         }
                    }
                }
            }

        }
        stage('pushing the helm chart to Nexus repo'){

            step{

                script{
                    withCredentials([string(credentialsId: 'admin_nexus_passwd', variable: 'nexus_creds')]) {
                        dir('kubernetes/') {
                          sh '''
                           helmversion=${helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ') }
                           curl -u admin:$nexus_creds http://3.84.125.50:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v'
                           '''
                        }
                    }
                }
            }
        }
    }  
    post{
        always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "testenv0103@gmail.com";  
		}
    }      
}    
