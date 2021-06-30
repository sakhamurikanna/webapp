pipeline {
    agent any
    tools { 
        maven 'Maven' 
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
            }
        }
	    
	    stage ('Check-Git-Secrets') {
		    steps {
	        sh 'rm trufflehog || true'
		sh 'docker pull gesellix/trufflehog'
		sh 'docker run -t gesellix/trufflehog --json https://github.com/devopssecure/webapp.git > trufflehog'
		sh 'cat trufflehog'
	    }
	    }
	    

	  stage ('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
	    
	 stage ('Source-Composition-Analysis') {
		steps {
		     sh 'rm owasp-* || true'
		     sh 'wget https://raw.githubusercontent.com/devopssecure/webapp/master/owasp-dependency-check.sh'	
		     sh 'chmod +x owasp-dependency-check.sh'
		     sh 'bash owasp-dependency-check.sh'
		     sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
		}
	}
	stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat-system']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war root@52.117.211.157:/var/lib/tomcat9/webapps/webapp.war'
              }      
           }       
        }
	stage ('SAST') {
		steps {
		withSonarQubeEnv('sonar') {
			sh 'mvn sonar:sonar'
			sh 'cat target/sonar/report-task.txt'
		       }
		}
	}
       
	 
	stage ('Port Scan') {
              steps {
			sh 'rm nmap* || true'
			sh 'docker run --rm -v "$(pwd)":/data uzyexe/nmap -sS -sV -oX nmap 52.117.211.157'
			sh 'cat nmap'
		    }
	    }
	stage ('DAST') {
		  
		    	steps {
			    sshagent(['zap']) {
				    sh 'ssh -o StrictHostKeyChecking=no ubuntu@65.2.124.2 "sudo docker run -t owasp/zap2docker-stable zap-baseline.py -t http://52.117.211.157:8080/webapp/" || true'
			    }
			}
		} 
        stage ('Nikto Scan') {
		    steps {
			sh 'rm nikto-output.xml || true'
			sh 'docker pull secfigo/nikto:latest'
			sh 'docker run --user $(id -u):$(id -g) --rm -v $(pwd):/report -i secfigo/nikto:latest -h 52.117.211.157 -p 8080 -output /report/nikto-output.xml'
			sh 'cat nikto-output.xml'   
		    }
	    }
	    
	 stage ('SSL Checks') {
		    steps {
			sh 'pip install sslyze'
			sh 'python -m sslyze --regular 0 devices / Proposed for decommission:8080 --json_out sslyze-output.json'
			sh 'cat sslyze-output.json'
		    }
	    }

     }	   
}	    	
