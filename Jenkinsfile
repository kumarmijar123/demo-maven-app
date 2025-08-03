pipeline {
   agent any
   environment {
   CONTAINER_REGISTRY_AND_REPO="kumarmijar3/mymavenapp"
}

     stages {
        stage('Checkout code') {
            steps {
                echo 'Cloning the repo'
            }
        }
         stage('Compile') {
            steps {
                echo 'Compiling the code'
                sh 'mvn -f pom.xml clean compile -DskipTest'
            }
        }
        stage('Unittest') {
          steps {
             echo "Running unitest"
             sh 'mvn -f pom.xml test'
}
}	
      	
         stage('Build and package app') {
            steps {
                sh 'mvn -f pom.xml package'
            }
             post {
  success {
        echo "Archiving the artifacts"
        archiveArtifacts artifacts: '**/*.war', followSymlinks: false, onlyIfSuccessful: true

        }
       }
    }
         	
         stage('Creating docker image') {
            steps {
                echo 'creating image'
                sh 'docker image build -t $CONTAINER_REGISTRY_AND_REPO:$BUILD_NUMBER .'
            }
        }
         stage('scan docker image') {
            steps {
                echo 'Scanning docker image'
               sh 'trivy image --format json --output result.json --scanners vuln --exit-code 1 --severity HIGH,CRITICAL --ignore-unfixed $CONTAINER_REGISTRY_AND_REPO:$BUILD_NUMBER'
            }
        }
         stage('Push docker image to registry') {
            steps {
            
                withDockerRegistry([credentialsId: 'dockerregcred', url: '']) {
                sh 'docker image push $CONTAINER_REGISTRY_AND_REPO:$BUILD_NUMBER'
            }
           
        }
}		
         stage('Deploy app to dev env') {
            steps {
                echo 'Deploying to dev env'
                sh 'docker container run -d --name myapp -p 8088:8080 $CONTAINER_REGISTRY_AND_REPO:$BUILD_NUMBER'
            }
        }
stage('Deploy to prod env') {
steps {
  echo "Deploying to prod. env"
  sh 'docker container run -d --name myapp-prod -p 8089:8080 $CONTAINER_REGISTRY_AND_REPO:$BUILD_NUMBER'   
    }
}

}
}
