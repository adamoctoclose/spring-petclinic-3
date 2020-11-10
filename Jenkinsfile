pipeline {
    //  parameters here provide the shared values used with each of the Octopus pipeline steps.
    parameters {
        // The space ID that we will be working with. The default space is typically Spaces-1.
        string(defaultValue: 'Spaces-203', description: '', name: 'SpaceId', trim: true)
        // The Octopus project we will be deploying.
        string(defaultValue: 'Pet Clinic', description: '', name: 'ProjectName', trim: true)
        // The environment we will be deploying to.
        string(defaultValue: 'Dev', description: '', name: 'EnvironmentName', trim: true)
        // The name of the Octopus instance in Jenkins that we will be working with. This is set in:
        // Manage Jenkins -> Configure System -> Octopus Deploy Plugin
        string(defaultValue: 'Octopus', description: '', name: 'ServerId', trim: true)
    }
    /*
        These are the tools we need for this pipeline. They are defined in Manage Jenkins -> Global Tools Configuration.
    */
    tools {
        maven 'maven 3'
    }
    agent any
    stages {

        stage('build') {
            steps {
                // Update the Maven project version to match the current build
                sh(script: "mvn versions:set -DnewVersion=1.0.${BUILD_NUMBER}", returnStdout: true)
                // Package the code
                sh(script: "mvn package -Pwar", returnStdout: true)
            }
        }
        stage('push packages') {
            steps {
                octopusPack additionalArgs: '', outputPath: "${env.WORKSPACE}/target", overwriteExisting: false, packageFormat: 'zip', packageId: 'petclinic.flyway', packageVersion: "1.0.${BUILD_NUMBER}", sourcePath: 'flyway', toolId: 'Default', verboseLogging: false
                octopusPushPackage additionalArgs: '', overwriteMode: 'FailIfExists', packagePaths: "${env.WORKSPACE}/target/petclinic.web.1.0.${BUILD_NUMBER}.war", serverId: "${ServerId}", spaceId: "${SpaceId}", toolId: 'Default'
                octopusPushPackage additionalArgs: '', overwriteMode: 'FailIfExists', packagePaths: "${env.WORKSPACE}/target/petclinic.flyway.1.0.${BUILD_NUMBER}.zip", serverId: "${ServerId}", spaceId: "${SpaceId}", toolId: 'Default'
                octopusPushBuildInformation additionalArgs: '', commentParser: 'GitHub', overwriteMode: 'FailIfExists', packageId: 'petclinic.web', packageVersion: "1.0.${BUILD_NUMBER}", serverId: "${ServerId}", spaceId: "${SpaceId}", toolId: 'Default', verboseLogging: false, gitUrl: 'main', gitCommit: "${GIT_COMMIT}"
            }
        }
        stage('deploy') {
            steps {
                octopusCreateRelease additionalArgs: '', cancelOnTimeout: false, channel: '', defaultPackageVersion: '', deployThisRelease: false, deploymentTimeout: '', environment: "${EnvironmentName}", jenkinsUrlLinkback: false, project: "${ProjectName}", releaseNotes: false, releaseNotesFile: '', releaseVersion: "1.0.${BUILD_NUMBER}", serverId: "${ServerId}", spaceId: "${SpaceId}", tenant: '', tenantTag: '', toolId: 'Default', verboseLogging: false, waitForDeployment: false
                octopusDeployRelease cancelOnTimeout: false, deploymentTimeout: '', environment: "${EnvironmentName}", project: "${ProjectName}", releaseVersion: "1.0.${BUILD_NUMBER}", serverId: "${ServerId}", spaceId: "${SpaceId}", tenant: '', tenantTag: '', toolId: 'Default', variables: '', verboseLogging: false, waitForDeployment: true
            }
        }
    }
}
