def COLOR_MAP = [
    "SUCCESS" : "good",
    "FAILURE"  : "danger",
]   
pipeline{
	agent "any"
	tools {
        maven "mvn"
        jdk "jdk-8"
    }	
    environment {
      SNAP_REPO = 'snap'
      NEXUS_USER = 'admin'
      NEXUS_PASS = 'admin123'
      RELEASE_REPO = 'repo-maven'
      CENTRAL_REPO = 'downloads'
      NEXUSIP = '13.40.78.4'
      NEXUSPORT = '8081'
      NEXUS_GRP_REPO = 'group'
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
        stage ('ANALYSIS WITH CHECKSTYLE'){
            steps {
               sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
        stage('CODE ANALYSIS with SONARQUBE'){
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
          }
        }
 
        stage("Quality Gate"){
            steps {
            timeout(time: 1, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
            }
        
          }
        }
  
     stage("Artifact uplaoder"){
      steps{
         nexusArtifactUploader(
        nexusVersion: 'nexus3',
        protocol: 'http',
        nexusUrl: '13.40.78.4:8081',
        groupId: 'QA',
        version: "",
        repository: 'repo-maven',
        credentialsId: 'nexus',
        artifacts: [
            [artifactId: "vproapp",
             classifier: '',
             file: 'target/vprofile-v2.war',
             type: 'war']
        ]
     )
    }
    }
   }

       post{
        always {
            echo 'Slack Notifications'
            slackSend channel: '#cicdproject',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
  }