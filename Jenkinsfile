!/usr/bin/env groovy

  pipeline {
    /////////////// IN THE label name, GIVE THE NAME OF THE PROJECT.                                                    /////////////////////
    /////////////// THE NAME MUST BE EXACTLY THE SAME AS THE GIVEN NAME in environment.PROJECT_NAME (LINE 56 ) VARABLE //////////////////////
    /////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
    agent {
      kubernetes {
        label "Jeu"
        yaml """
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      component: ci
  spec:
    serviceAccountName: jenkins
    containers:
      - name: docker
        image: docker
        command:
          - cat
        tty: true
        volumeMounts:
          - mountPath: /var/run/docker.sock
            name: docker-sock
      - name: kubectl
        image: lachlanevenson/k8s-kubectl:v1.17.4 
        command:
          - cat
        tty: true
    volumes:
      - name: docker-sock
        hostPath:
          path: /var/run/docker.sock

  """
      }
    }

  //////////////////////////////////////////////////////////////////////////////
  ////////////    CHANGE ONLY THE VARABLES IN environmant tag ///////////////////
  //////////////////////////////////////////////////////////////////////////////
    environment {
      PROJECT_NAME="Jeu"          // project name with no any space
      IMAGE_TAG='1.0'               // the tag or version of docker image
      DEPLOYMENT_ENV='development'    // deployment environment in kubernetes cluster
    }

  //////////////////////////////////////////////////////////////////////////////
  /////////////////  END OF CHANGABLE VARABLE    ///////////////////////////////
  //////////////////////////////////////////////////////////////////////////////
    tools {nodejs "node"}
    
      stages {

        stage('Sonarqube') {
        environment {
            scannerHome = tool 'SonarQubeScanner'
        }
        steps {
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner"
            }
        }
    }

        stage ('Build image') {
          steps {
            container('docker') {
              sh "docker build -t registry.rintio.com:5000/${PROJECT_NAME}:${IMAGE_TAG} ."
              sh "docker push registry.rintio.com:5000/${PROJECT_NAME}:${IMAGE_TAG}"
            }
          }
        }
        
        stage('Test projet'){
          steps {
            container('maven'){
             sh "mvn test"
            }
          }
        }
        
        stage('Scripting') {
          steps {
              sh 'chmod a+x deploy_script.sh'
              sh './deploy_script.sh'
          }
          post {
                always {
                    junit 'target/surefire-reports/*.xml' 
                }
          }
        }
  } 
