pipeline {
    
	agent any

	tools {
        maven "mvn"
        jdk "jdk-8"
    }	
  
      environment {
      SNAP_REPO = 'maven-snap'
      NEXUS_USER = 'admin'
      NEXUS_PASS = 'admin123'
      RELEASE_REPO = 'maven-hosted'
      CENTRAL_REPO = 'maven-proxy'
      NEXUSIP = '172.31.39.81'
      NEXUSPORT = '8081'
      NEXUS_GRP_REPO = 'maven-group'
      NEXUS_LOGIN = 'nexus'
      SONARSERVER = "sonar-server"
      SONARSCANNER = "sonar-pro"
    }
  stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn clean install'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
/* 
	stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

	stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
	
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
*/ 
        stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool 'sonar-pro'
          }

          steps {
            withSonarQubeEnv('sonar-server') {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }


            timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
            }
         
          }
        }
  }
}

    



