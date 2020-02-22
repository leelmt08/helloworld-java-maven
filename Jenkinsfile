node {
    def server = Artifactory.server 'A3'
    //def rtGradle = Artifactory.newGradleBuild()
    def buildInfo


  stage('SCM Checkout'){
	git  'https://github.com/leelmt08/helloworld-java-maven'
   }
   
     stage('SonarQube Analysis') {
        def mvnHome =  tool name: 'mvn3', type: 'maven'
        withSonarQubeEnv('sonar7.5') { 
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
	     
    
//      stage('Validate stage') {
// 		def mvnHome = tool name: 'mvn3', type: 'maven'
//                 bat "${mvnHome}/bin/mvn validate"
//         }
        
try {
    stage ('Tesing Stage') {
       //rtMaven.run pom: 'pom.xml', goals: 'test'
        //def mvnHome = tool name: 'mvn3', type: 'maven'
        bat("mvn test")
    } } catch (err) {
        mail bcc: '', body: "${err}", cc: '', from: '', replyTo: '', subject: 'Failed build', to: 'leebhovi@gmail.com'
        echo "${err}"  
    }


     stage('Results') {
            junit '**/target/surefire-reports/TEST-*.xml'
            //archive 'target/*.jar'
            archiveArtifacts artifacts: 'screenshots/**,build/test/results/*.xml', allowEmptyArchive: true
         }


    //stage('Email Notification') {
	//	mail bcc: '', body: 'test success', cc: '', from: '', replyTo: '', subject: 'jenkins-test', to: 'leebhovi@gmail.com'
	//	 }
 
 	stage ('Compile package'){
	//Get mvn home path
	//def mvnHome = tool name: 'mvn3', type: 'maven'
		bat "mvn package"
	}
	
   // stage ('Deploy') {
     //   rtMaven.deployer.deployArtifacts //buildInfo
    //}
         stage ('Artifactory Config') {
        // Obtain an Artifactory server instance, defined in Jenkins --> Manage:
        server = Artifactory.server 'A3'

        rtMaven = Artifactory.newMavenBuild()
        rtMaven.tool = 'mvn3' // Tool name from Jenkins configuration
        rtMaven.deployer releaseRepo: 'art-example', snapshotRepo: 'art-snapshot', server: server
        rtMaven.resolver releaseRepo: 'art-example', snapshotRepo: 'art-snapshot', server: server
        rtMaven.deployer.deployArtifacts = false // Disable artifacts deployment during Maven run
        buildInfo = Artifactory.newBuildInfo()
    }
    
    stage ('Deploy') {
    rtMaven.deployer.deployArtifacts //buildInfo     
    //rtMaven.deployer.deployArtifacts = true
    buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean compile install -DskipTests=true', buildInfo: buildInfo;
    }
    
    stage ('Publish build info') {
        server.publishBuildInfo buildInfo
    }
}
