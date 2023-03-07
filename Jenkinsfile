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
    }        
}    
