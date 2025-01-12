pipeline {
  agent any
  tools {
  maven 'Maven'
  }
    
	stages {

      stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/DevOps-Module/2024_july-bootcamp.git']]])
        }
      }
	  
	  stage ('Build')  {
	      steps {
            dir('webapp'){
            sh "pwd"
            sh "ls -lah"
            sh "mvn package"
          }
        }   
      }
   
     stage ('SonarQube Analysis') {
        steps {
              withSonarQubeEnv('sonar') {
				dir('webapp'){
                 sh 'mvn -U clean install sonar:sonar'
                }		
              }
            }
      }

    stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "jfrog",
                    url: "http://35.176.16.122:8082/artifactory", 
                    credentialsId: "jfrog"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog",
                    releaseRepo: "july24-libs-release-local", 
                    snapshotRepo: "july24-libs-snapshot-local" 
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog", // credential ID from Jenkins global credentials
                    releaseRepo: "july24-libs-release-local",
                    snapshotRepo: "july24-libs-snapshot-local"
                )
            }
    }

    //stage ('Deploy Artifacts') {
    //        steps {
    //            rtMavenRun (
    //                tool: "Maven", // Tool name from Jenkins configuration
    //                pom: 'webapp/pom.xml',
    //                goals: 'clean install',
    //                deployerId: "MAVEN_DEPLOYER",
    //                resolverId: "MAVEN_RESOLVER"
    //            )
    //     }
   // }

    stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "jfrog"
             )
        }
    }

    stage('Copy Dockerfile & Playbook to Staging Server') {
            
            steps {
                  sshagent(['ssh_agent']) {
                       sh "chmod 400 mo-london-kp.pem" 
                       sh "ls -lah"
                        sh "scp -i mo-london-kp.pem -o StrictHostKeyChecking=no dockerfile ubuntu@18.171.23.53:/home/ubuntu"
                        sh "scp -i mo-london-kp.pem -o StrictHostKeyChecking=no push-2-dockerhub.yaml ubuntu@18.171.23.53:/home/ubuntu"
                    }
                }
        } 

    stage('Build Container Image') {
            
            steps {
                  sshagent(['ssh_agent']) {
                        sh "ssh -i mo-london-kp.pem -o StrictHostKeyChecking=no ubuntu@18.171.23.53 -C \"ansible-playbook -vvv -e build_number=${BUILD_NUMBER} push-2-dockerhub.yaml\""       
                    }
                }
        } 

    stage('Copy Deployment & Service Defination to K8s Master') {
            
            steps {
                  sshagent(['ssh_agent']) {
                        sh "scp -i mo-london-kp.pem -o StrictHostKeyChecking=no deploy_service.yaml ubuntu@35.178.248.53:/home/ubuntu"
                        }
                }
        } 

    stage('Waiting for Approvals') {
            
        steps{
		input('Test Completed ? Please provide Approvals for Prod Release ?')
			 }
    }     
    stage('Deploy Artifacts to Production') {
            
            steps {
                  sshagent(['ssh_agent']) {
                        //sh "ssh -i mo-oregon-kp.pem -o StrictHostKeyChecking=no ubuntu@52.27.146.236 -C \"kubectl set image deployment/ranty customcontainer=mobanntechnologies/july-set:${BUILD_NUMBER}\"" 
                        //sh "ssh -i mo-oregon-kp.pem -o StrictHostKeyChecking=no ubuntu@18.135.243.72 -C \"kubectl delete deployment ranty && kubectl delete service ranty\""
                        sh "ssh -i mo-london-kp.pem -o StrictHostKeyChecking=no ubuntu@35.178.248.53 -C \"kubectl apply -f deploy_service.yaml\""
                        //sh "ssh -i mo-oregon-kp.pem -o StrictHostKeyChecking=no ubuntu@52.27.146.236 -C \"kubectl apply -f service.yaml\""
                    }
                }  
        } 
   } 
}


