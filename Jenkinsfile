node {
    def server = Artifactory.server 'A3'
    def buildInfo
    def rtMaven

  stage('SCM Checkout'){
	git  'https://github.com/leelmt08/helloworld-java-maven'
   }
   
        stage('SonarQube Analysis') {        
        def mvnHome =  tool name: 'mvn3', type: 'maven'        
        withSonarQubeEnv('sonar7.5') 
            {           
                bat "${mvnHome}/bin/mvn sonar:sonar"        
            }    
        }
        stage("Quality Gate Status Check"){
          timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
          }
      } 

    
try {
    stage ('Tesing Stage') {
       //rtMaven.run pom: 'pom.xml', goals: 'test'
        //def mvnHome = tool name: 'mvn3', type: 'maven'
        bat("mvn test")
    } } catch (err) {
        mail bcc: '', body: "${err}", cc: '', from: '', replyTo: '', subject: 'Failed build', to: 'leebhovi@gmail.com'
        echo "${err}"  
    }
    

         stage('Test Results') {
            junit '**/target/surefire-reports/TEST-*.xml'
            //archive 'target/*.jar'
            archiveArtifacts artifacts: 'screenshots/**,build/test/results/*.xml', allowEmptyArchive: true
         }

    
   stage ('Artifactory configuration') {
        // Obtain an Artifactory server instance, defined in Jenkins --> Manage:
        server = Artifactory.server 'A3'

        rtMaven = Artifactory.newMavenBuild()
        rtMaven.tool = 'mvn3' // Tool name from Jenkins configuration
        rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local', server: server
        rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot', server: server
        rtMaven.deployer.deployArtifacts = false // Disable artifacts deployment during Maven run

        buildInfo = Artifactory.newBuildInfo()
    }
 
    stage ('Clean Test') {
        rtMaven.run pom: 'pom.xml', goals: 'clean test'
    }
        
    stage ('Install') {
        rtMaven.run pom: 'pom.xml', goals: 'install', buildInfo: buildInfo
    }
 
    stage ('Deploy') {
        rtMaven.deployer.deployArtifacts buildInfo
    }
        
    stage ('Publish build info') {
        server.publishBuildInfo buildInfo
    }
}

