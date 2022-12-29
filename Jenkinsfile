 //on windows system, bat should replace sh(unix)

def VERSION_SUIVANTE
def VERSION_ACTUELLE
def VERSION_DEFAULT
def IMAGE
def pom
//preprod => test d endurance, technique ...
pipeline {
    //agent { label "any-executor" }
    agent any
    
    // global environment
    //environment{
        //--maven will be used from here --//
        //PATH = /opt/apache-maven-xxxx/bin:$PATH  <= MANUAL MAVEN PAH SETTING
    //}
    tools { 
        maven 'maven'
        jdk 'java-11'
    }
    parameters {
        choice(name: 'ENVIRONMENT',
            choices: 'DEV\nTEST\nPREPROD-UAT-ACCEPTANCE\nPROD',
            description: 'Please select the Build Environment')
    }

    stages {

        stage ('Initialize') {
            //specific environment
            //environment{
              //PATH = /opt/apache-maven-xxxx/bin:$PATH  <= MANUAL MAVEN PAH SETTING
            //}
            steps {
              echo "The buildEnv is ${params.ENVIRONMENT.toLowerCase()}"
              sh '''
                git status
                ls .
              '''
              echo '----############################################----------'
               
              script {
                init()
              }
            }
        }
        stage('Build') {
            steps {
                script {
                    build()
                }
                echo "---------------  clean compile package -------------------"

                sh "mvn clean package"
                //withSonarQubeEnv('sonarLocal') {
                  //  sh "mvn -DskipTests sonar:sonar"
                //}
            }

        }
        stage('Deploy') { 
            steps {
                //deploy to nexus
                retry(3) {
                     echo "---------- deploy to nexus ----->  mvn -DskipTests deploy -------------------"
                    //sh "mvn -DskipTests deploy" this will use the deplyment setting from pom.xml
                    // there is a way using the plugin nexus
                }
            }
        }
        stage('Deliver') {
            parallel{
                stage('Deliver1') {
                    steps {
                        echo 'Deliver1'
                        script {
                            echo 'deliver'
                            // deliver()
                        }
                        
                    }
                    post {
                        always {
                            echo 'After deploy to tomcat'
                            script {
                                echo 'afterDeliver'
                                // afterDeliver()
                            }
                        }
                    }
                }
                stage('Deliver2') {
                    steps {
                        echo "-------------------push to git 2-------------------------"
                    }
                }
            } 
        }
    }
    post {
        always {
            echo 'I have finished and deleting workspace'
            //deleteDir() 
        }
        success {
            echo 'Job succeeeded!'
        }
        unstable {
            echo 'I am unstable :/'
        }
        changed {
            echo 'Things were different before...'
        }

        failure {
            echo 'Job failed :('
        }
        aborted {
            echo 'Has beenn aborted'
        }  
    }
}



/////
def init() {
    pom = readMavenPom file: 'pom.xml'  // library pipeline-utility-steps to be install

    IMAGE = readMavenPom().getArtifactId()
    VERSION_DEFAULT = readMavenPom().getVersion()
    echo '--------- GET VERSION ------------'
    // VERSION_DEFAULT = pom.version
    echo '--------- VERSION_DEFAULT ------------'
    echo VERSION_DEFAULT
    echo '--------- ARTIFACT ------------'
    echo IMAGE
    echo '--------- AGAIN ------------'
    echo "IMAGE: ${IMAGE}"
    echo "VERSION: ${VERSION_DEFAULT}"
    echo '--------- AGAIN END------------'
    if( params.ENVIRONMENT == 'PREPROD'){
        timeout(time: 30, unit: 'SECONDS') {

            def VERSION_TAB
            VERSION_ACTUELLE = VERSION_DEFAULT.replaceAll('-SNAPSHOT','').trim()
            //pom = '9.98.98-SNAPSHOT'
            //VERSION_ACTUELLE = pom.replaceAll('-SNAPSHOT','').trim()
            echo '-----------------------'
            VERSION_TAB = VERSION_ACTUELLE.split('\\.')
            echo 'instance of string[]?'
            println VERSION_TAB instanceof String[]
            println VERSION_TAB.size()
            echo '-----------------------'
         
            for(int i = VERSION_TAB.size()-1; i>=0; i--) {
               
                 println(i);
                 echo VERSION_TAB[i]
                 lastitem = VERSION_TAB[i]
                 
                 def nextvalue
                 if (lastitem.isInteger()) {
                     lastitemint = lastitem as Integer
                     nextvalue = lastitemint + 1
                 }
                 echo 'nextvalue'
                 println nextvalue
                 
                 if(i == 0){
                     echo 'on met a jours la seule valeur'
                     VERSION_TAB[i] = nextvalue
                     break
                 }
                 
                 retenue = ( nextvalue%100 == 0) ? 1:0
                 if(retenue == 1){
                     echo 'la next value est un multiple de 100'
                     nextvalue = 0
                     VERSION_TAB[i] = nextvalue
                 }else{
                     echo 'la next value n est pas un multiple de 100'
                     VERSION_TAB[i] = nextvalue
                     break
                 }
            }
            echo '--------4---------'
            VERSION_SUIVANTE = VERSION_TAB.join('.')
            echo 'la version siuvante est:'
            echo VERSION_SUIVANTE
         
            // Show the select input modal
            def INPUT_PARAMS = input message: 'Please Provide Parameters', ok: 'Next',
                parameters: [
                    string(name: 'version_actuelle',
                        defaultValue: VERSION_ACTUELLE,
                        description: 'current build version'),
                    string(name: 'version_suivante',
                        defaultValue: VERSION_SUIVANTE,
                        description: 'next build version')
                ]

            env.version_suivante = INPUT_PARAMS.version_suivante
            env.version_actuelle = INPUT_PARAMS.version_actuelle

        }
    }else{
        env.version_default = VERSION_DEFAULT
    }
}

def build() {
  if( params.ENVIRONMENT == 'PREPROD'){
      echo "---------- upadate pom version to ----->  ${env.version_actuelle} -------------------"
      sh "mvn versions:set -DnewVersion=${env.version_actuelle}"
  }
}

def deliver() {
    if( params.ENVIRONMENT == 'PREPROD'){
        echo "----- deploy to tomcat ------------ vesion --> ${env.version_actuelle} -------------"
     
      //-----------1---------Local windows manuel deploiement------------------------------------------------
        /*
           powershell 'build.ps1'            
           powershell ("""
             xcopy C:\\Users\\merlin\\.jenkins\\workspace\\testPipeline\\target\\nexus-${env.version_actuelle}.war C:\\Tomcat\\tomcat-8\\apache-tomcat-8.5.16\\webapps /s /y
             cd C:\\Tomcat\\tomcat-8\\apache-tomcat-8.5.16\\bin
             dir .
             & .\\catalina.bat
           """)
        */
      //-------fin-----Local windows manuel deploiement------------------------------------------------
     
      //---------2---------------REMOTE deploiement------------use SSH------------from ssh agent plugin------------------------
      sshagent(['deploy_tomcat']) {
        // scp <src_file> server_username@IP:<dest_file>
        // to avoid KeyChecking: -o StrictHostKeyChecking=no
        //sh "scp -o StrictHostKeyChecking=no ./target/nexus-${env.version_default}.war ubuntu@18.222.143.128:/opt/tomcat/webapps"
        
        sh '''
           scp -o StrictHostKeyChecking=no ./target/*.war ubuntu@18.222.143.128:/home/ubuntu
           ssh -o StrictHostKeyChecking=no ubuntu@18.222.143.128 'sudo chown -R ubuntu:ubuntu /opt/tomcat/'
           ssh -o StrictHostKeyChecking=no ubuntu@18.222.143.128 'cp -r /home/ubuntu/*.war /opt/tomcat/webapps/'
           ssh -o StrictHostKeyChecking=no ubuntu@18.222.143.128 'sudo chown -R tomcat:tomcat /opt/tomcat/'
        '''
       
      }
      //---------3----------------------Or using deploiement plugin from jenkins----------------------------
    }else{
      echo "----- deploy to tomcat ------------ vesion --> ${env.version_default} -------------"
     
      //---------1-------------------------Local windows manuel deploiement------------------------------------------------
      //----Avec powershell----
      /*
         powershell ("""
            xcopy .\\target\\nexus-${env.version_default}.war C:\\Tomcat\\tomcat-7\\apache-tomcat-7.0.67\\webapps /s /y
            cd C:\\Tomcat\\tomcat-8\\apache-tomcat-8.5.16\\bin
            dir .
            & .\\catalina.bat
         """)
       */
      //-------fin-----Local windows manuel deploiement------------------------------------------------
      //---------2-------------------------------------Avec bat/sh et curl -----------------------------------
      //bat "dir .\\target\\" or ls ./target/
      //def response = bat or ssh (script: "curl -s --upload-file .\\target\\nexus-${env.version_default}.war http://deployer:deployer@localhost:8083/manager/text/deploy?path=/nexus", returnStdout: true)
      //echo response
     
      //------3-----------REMOTE deploiement------------use SSH------------from ssh agent plugin------------------------
      echo "-------REMOTE deploiement------------use SSH------------from ssh agent plugin--------------"
      sshagent(['deploy_tomcat']) {
        // scp <src_file> server_username@IP:<dest_file>
        // to avoid KeyChecking: -o StrictHostKeyChecking=no
        //sh "scp -o StrictHostKeyChecking=no ./target/nexus-${env.version_default}.war ubuntu@18.222.143.128:/opt/tomcat/webapps"
        
        sh '''
           scp -o StrictHostKeyChecking=no ./target/*.war ubuntu@18.222.143.128:/home/ubuntu
           ssh -o StrictHostKeyChecking=no ubuntu@18.222.143.128 'sudo chown -R ubuntu:ubuntu /opt/tomcat/'
           ssh -o StrictHostKeyChecking=no ubuntu@18.222.143.128 'cp -r /home/ubuntu/*.war /opt/tomcat/webapps/'
           ssh -o StrictHostKeyChecking=no ubuntu@18.222.143.128 'sudo chown -R tomcat:tomcat /opt/tomcat/'
        '''
       
      }
      //-------4-----------Or using deploiement plugin from jenkins
    }
}

def afterDeliver() {
  if( params.ENVIRONMENT == 'PREPROD'){
      echo "---------- upadate pom version to ----->  ${env.version_suivante} -------------------"
      sh "mvn versions:set -DnewVersion=${env.version_suivante}-SNAPSHOT"
      echo "push to git"
      //sshagent (credentials: ['7afc0e25-f6dd-41d5-97a5-1422d961386f']){
         //sh '''
            //git show-ref
            //git config --global user.name "merlin"
            //git config --global user.email "merlin@gmail.com"
            //git status
            //git add .
            //git status
            //git commit pom.xml -m "Bumped version number"
            //git push -u origin HEAD:master
         //'''
      //}
      //withCredentials([usernamePassword(credentialsId: '7afc0e25-f6dd-41d5-97a5-1422d961386f', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USER')]) {
      withCredentials([usernamePassword(credentialsId: '7afc0e25-f6dd-41d5-97a5-1422d961386f', passwordVariable: 'GIT_USER', usernameVariable: 'GIT_PASSWORD')]) { 
         //sh '''
            //git show-ref
            //git config --global user.name "merlin"
            //git config --global user.email "merlin@gmail.com"
            //git status
            //git add .
            //git status
            //git commit pom.xml -m "Bumped version number"
            //git push -u origin HEAD:main
         //'''
         def encodedPassword = URLEncoder.encode("$GIT_PASSWORD",'UTF-8')
         // https://github.com/TTM-Developers/testPipe.git
         //git push -u origin HEAD:main
         //git push https://${GIT_USER}:${encodedPassword}@github.com/${GIT_USER}/testPipe.git
         sh '''
            git config user.name merlin
            git config user.email merlin@gmail.com
            git add .
          '''
          sh "git commit -m 'Triggered Build: ${env.version_suivante}'"
          //sh "git push -u origin HEAD:main https://${GIT_USER}:${encodedPassword}@github.com/TTM-Developers/testPipe.git"
          sh "git push -u origin HEAD:main"
         
      }
  }else{

  }
}
