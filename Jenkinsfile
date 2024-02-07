pipeline{

        agent {
                label "Pipeline-slave"
                }
                stages{
                        stage ("Pull the code latest from SCM"){
                                steps {
                                        git branch: 'main', url: 'https://github.com/gouravaas/new_java_docker_app.git'
                                        }
                                }
                        stage (" Build the code "){
                                steps {
					sh 'sudo mvn dependency:purge-local-repository'
                                        sh 'sudo mvn clean package'
                                        }
                                }
			stage (" Build a docker Image " ){
			     	steps {
					sh 'sudo docker build -t new-java-app:$BUILD_TAG .'
					sh 'sudo docker tag new-java-app:$BUILD_TAG gouravaas/new-java-app:$BUILD_TAG'
				}
			}
			stage (" Push to dockerhub" ){
				steps {
					withCredentials([usernamePassword(credentialsId: 'Docker_hub', passwordVariable: 'docker_hub_pass_var', usernameVariable: 'docker_hub_user_var')]) {
					sh 'sudo docker login -u ${docker_hub_user_var} -p ${docker_hub_pass_var}'
					sh 'sudo docker push gouravaas/new-java-app:$BUILD_TAG'
}
					}
	
			}
			stage (" Testing the pipeline" ){
				steps {
					sh 'sudo docker run -dit -p 8080:8080 new-java-app:$BUILD_TAG'
				}
			}
			stage("testing website") {
				steps {
					retry(5) {
						script {
							sh 'curl --silent http://172.31.43.161:8080/java-web-app/ | grep -i -E "(india|sr)" '
							}
						}
					}				
			}
			stage("Approval status") {
				steps {
					script {
						 Boolean userInput = input(id: 'Proceed1', message: 'Do you want to Promote this build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                				echo 'userInput: ' + userInput
					}
				}
			}
                	stage("Prod Env") {
				steps {
					sshagent(['production']) {
			    	 	sh "ssh -o StrictHostKeyChecking=no ubuntu@54.88.198.4 sudo docker run  -d  -p  49153:8080  gouravaas/new-java-app:$BUILD_TAG"
				}
			}
		}
		}
        }


