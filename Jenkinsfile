pipeline {
  agent any 
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                echo "PATH = ${PATH}"
                echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
     }
    
    stage ('Check secrets') {
      steps {
      sh 'trufflehog3 https://github.com/purva1708/prpdevsecops.git -f json -o truffelhog_output.json || true'
      }
    }
  
      stage ('Software composition analysis') {
            steps {
                dependencyCheck additionalArguments: ''' 
                    -o "./" 
                    -s "./"
                    -f "ALL" 
                    --prettyPrint''', odcInstallation: 'owasp-dc'

                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }

  
   stage ('SAST - SonarQube') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'mvn clean sonar:sonar -Dsonar.java.binaries=src'
	  //sh 'sudo python3 sonarqube.py'
	 // sh './sonarqube_report.sh' 
        }
      }
    } 
  


  
    stage ('Generate build') {
      steps {
        sh 'mvn clean install -DskipTests'
      }
    } 

    stage ('Deploy to server') {
            steps {
	   timeout(time: 10, unit: 'MINUTES') {
              sshagent(['app-server']) {
                sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/devsecops/webgoat-server/target/webgoat-server-v8.2.0-SNAPSHOT.jar ubuntu@13.232.36.252:/WebGoat'
		sh 'ssh -o  StrictHostKeyChecking=no ubuntu@13.232.36.252 "nohup java -jar /WebGoat/webgoat-server-v8.2.0-SNAPSHOT.jar --server.address=13.232.36.252 --server.port=9999 &"'
                  }
	     }
        }     
    }

   stage ('DAST - OWASP ZAP') {
            steps {
           sshagent(['dast-server']) {
		 'ssh -o  StrictHostKeyChecking=no apps@10.97.109.243 "sudo docker run --rm -v /home/apps:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://13.232.36.252:8080/WebGoat -x zap_report || true" '
               //sh 'ssh -o  StrictHostKeyChecking=no ubuntu@65.0.176.6 "sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://13.232.36.252:9999/WebGoat -x zap_report || true" '
		//sh 'ssh -o  StrictHostKeyChecking=no apps@10.97.109.243 "sudo docker run --rm -v /home/apps:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://10.97.109.244:8081/WebGoat -x zap_report -n defaultcontext.context || true" '
	      // sh 'ssh -o  StrictHostKeyChecking=no ubuntu@65.0.176.6 "./zap_report.sh"'
              }      
           }       
    }
  }
}

