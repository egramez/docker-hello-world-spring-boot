node {
    // reference to maven
    // ** NOTE: This 'maven-3.6.1' Maven tool must be configured in the Jenkins Global Configuration.   
    def mvnHome = tool 'maven-3.6.1'

    // holds reference to docker image
    def dockerImage
    // ip address of the docker private repository(nexus)
    
    def dockerRepoUrl = "pickmeacr.azurecr.io"
    def dockerImageName = "dev/demo-app"
    def dockerImageTag = "${dockerRepoUrl}/${dockerImageName}:${env.BUILD_NUMBER}"
    
    stage('Clone Repo') 
     { 
       node("build-server") {
       // for display purposes
      // Get some code from a GitHub repository
      git 'https://github.com/egramez/docker-hello-world-spring-boot.git'
      // Get the Maven tool.
      // ** NOTE: This 'maven-3.6.1' Maven tool must be configured
      // **       in the global configuration.           
      mvnHome = tool 'maven-3.6.1'
       }
    }    
  
    stage('Build Project') {
      // build project via maven
             node("build-server") {
      sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
             }
    }
	
	stage('Publish Tests Results'){
                 node("build-server") {
      parallel(
        publishJunitTestsResultsToJenkins: {
          echo "Publish junit Tests Results"
		  junit '**/target/surefire-reports/TEST-*.xml'
		  archive 'target/*.jar'
        },
        publishJunitTestsResultsToSonar: {
          echo "This is branch b"
      })
                 }
    }
		
    stage('Build Docker Image') {
      node("build-server") {
      // build docker image
      sh "whoami"
      sh "ls -all /var/run/docker.sock"
      sh "mv ./target/hello*.jar ./data" 
      
      sh "docker build . -t ${dockerImageTag}"


      //dockerImage = docker.build("hello-world-java:${env.BUILD_ID}")
      }
    }
   
    stage('Publish Docker Image'){
      node("build-server") {
      // deploy docker image to nexus
      withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'acrcred',
      usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {

      echo "Docker Image Tag Name: ${dockerImageTag}"

      sh "docker login -u $USERNAME -p $PASSWORD  ${dockerRepoUrl}"
      //sh "docker tag ${dockerImageName} ${dockerImageTag}" 
      sh "docker push ${dockerImageTag}"      
      }

    }
 }

}
