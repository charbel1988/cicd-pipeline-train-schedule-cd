pipeline {
    agent any
    stages {
         stage('Build') {
            steps {
                echo 'Running build automation' //echo some text on the screen
                sh './gradlew build --no-daemon' //turn off the gradle daemon and run the build automation gradlew
                archiveArtifacts artifacts: 'dist/trainSchedule.zip' //archive the application in a deployable zip file
            }
        }
           stage('DeployToStaging') {
            when {
                branch 'master' //this will deplpoy only when code is committed in the master branch
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) { //this binds the crednetials with this block so we can access them. Referencing the webserver_login account in Jenkins
                    sshPublisher( //this allows the servers to move files over ssh
                        failOnError: true,
                        continueOnError: false,
                        publishers: [ //define a set of publishers
                            sshPublisherDesc(
                                configName: 'staging', //the name of the plugin config which is referenced to the configname of the public ssh server we added
                                sshCredentials: [ // set of credentials which are referenced in the with credentials block
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip', //transfer the files from the build stage to move it to the staging server
                                        removePrefix: 'dist/', //remove the prefix dist so it doesn't create the new direcotry as dist in the linux box
                                        remoteDirectory: '/tmp', //this is where the zip file will be added to on the staging server
                                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule' //issue a exec command on the linux staging server to deploy the zip app in opt.
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Does the staging environment look OK?' //same as deploying to Production except we had the prompt for the user to proceed with the Production deployment
                milestone(1) //if i approve a particular deployment than this will prevent any previous builds from running. It will deploy only the new version of the code.
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'production',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}
