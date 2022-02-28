pipeline {
        
	agent { label 'agent1' }
        
	environment {
	        boolean passedtest = false
                CHANGE_ID = "${env.CHANGE_ID}"
                boolean tests = false
	}
        
	stages {
                
                stage('ENVs'){
                        steps {
                                sh 'printenv'
                                sh 'echo "branch PR-${CHANGE_ID}"'
                        }
                }
                
		stage('Tests') {
			when { 
                                environment name: 'GIT_BRANCH', value: "PR-${CHANGE_ID}"
                        }
                        steps {   
                            echo 'tests performing'
			    script {
                                passedtest = true
                                tests = true
			        try {
                                        sh 'mvn clean package -Dcheckstyle.skip -Dspring.profiles.active=PostgreSQL'
                                } catch (Exception e) {
                                        passedtest = false
                                }
			    }
		        }
                }

		stage('Build') {
			when { branch 'main' }
                        
			steps { 
                                sh '''
                                git checkout main
                                git config --global user.email "jenkins@example.com"'
                                git config --global user.name "agent1"'
                                
                                mvn -B release:prepare -DpushChanges=false'''

                                sh 'mvn release:perform -Darguments="-Dmaven.javadoc.skip=true"' 
                                
                        } 
		}
	}      
		post {
                        always {
                             script {
                                     if (passedtest==true) {
                                   withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
 					sh '''
                                        git branch -r
					git checkout main
					git merge origin/PR-${CHANGE_ID-0}
					git push origin main
                                        '''
                                   }
				      } else {
                                                if (tests==true) {
                                                        echo 'tests failed' 
                                             } else { echo 'tests skipped' }
                                        }
                                
                                     }
                        }
                        failure {

                                 sh 'curl -X POST -d "type=alert&content=job failed" https://192.168.0.111:80/'

                         }
                        cleanup {
                                cleanWs() 
                        }
		}
       
}
