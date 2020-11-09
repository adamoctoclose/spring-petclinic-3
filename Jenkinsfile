pipeline {
    agent any
    tools {
        maven 'maven 3'
        jdk 'jdk'
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

        stage ('Build') {
            steps {
                sh 'mvn versions:set -DnewVersion=3.0.${BUILD_NUMBER}'
                sh 'mvn package -Pwar'
            }
        }
        
        stage ('Pack Flyway') {
            steps {
                    sh """
                        echo ""
                        ${tool('Octo CLI')}octo pack --id petclinic.flyway --format zip --version 3.0.${BUILD_NUMBER} --outFolder target --basePath flyway                   
                    """
                }
            }
        
        stage ('Push to Octopus') {
            steps {
                withCredentials([string(credentialsId: 'OctopusAPIKey', variable: 'APIKey')]) {
                    sh """
                        ${tool('Octo CLI')}octo push --package target/petclinic.3.0.${BUILD_NUMBER}.war --replace-existing --server https://samples.octopus.app --apiKey ${APIKey} --space Spaces-203
                        ${tool('Octo CLI')}octo push --package target/petclinic.flyway.3.0.${BUILD_NUMBER}.zip --replace-existing --server https://samples.octopus.app --apiKey ${APIKey} --space Spaces-203                       
                    """
                }
            }
        }
    }
}
