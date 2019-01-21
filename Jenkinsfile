properties([pipelineTriggers([githubPush()])])

node{
     
    def projectName      = "jenkins-training"
    def slackToken       = "${env.SLACK_PLATFORMS_ALERTS}"
    def credentialsId = "${env.SSH_CREDENTIALS_ID}"

     stage("Checkout"){
            echo "git checkout"
            checkout changelog: false, poll: false, scm: [
                $class: 'GitSCM',
                branches: [[
                    name: 'master'
                ]],
                doGenerateSubmoduleConfigurations: false,
                extensions: [[
                    $class: 'WipeWorkspace'
                ], [
                    $class: 'CleanBeforeCheckout'
                ]],
                submoduleCfg: [],
                userRemoteConfigs: [[
                    url: "git@github.com:telegraph/${projectName}.git"
                ]]
            ]
            } 
     stage("Webserver Build") {
        sh """
           #!/bin/sh
           if aws --profile preprod cloudformation --region eu-west-1 describe-stacks \
                  --stack-name ${projectName}-${username}; then
                  aws --profile preprod cloudformation --region eu-west-1 update-stack --stack-name ${projectName}-${username} \
                      --template-body file://infrastructure/templates/webserver.yaml \
                      --capabilities CAPABILITY_IAM \
                      --parameters file://infrastructure/parameters/parameters-webserver.json
           else
                  aws --profile preprod cloudformation --region eu-west-1 create-stack --stack-name ${projectName}-${username} \
                      --template-body file://infrastructure/templates/webserver.yaml \
                      --capabilities CAPABILITY_IAM \
                      --parameters file://infrastructure/parameters/parameters-webserver.json
            fi
            """
    }

     stage("Build Triggered Notify Slack"){
            gitCommit = sh(returnStdout: true, script: "git log -1 --pretty=format:'%h %an %s'").trim()
            gitFullSha1 = sh(returnStdout: true, script: "git rev-parse HEAD").trim()

            slackSend message: "${projectName} - started build to preprod. Most recent commit: \n${gitCommit}(<https://github.com/telegraph/${projectName}/commits/${gitFullSha1}|Open>)", token: "${env.SLACK_PLATFORMS_ALERTS}", channel: "${slack_id}", teamDomain: "telegraph", baseUrl: "https://hooks.slack.com/services/", color: "good"
    }

     stage("Get PublicIp"){
      sh """
          while [ "\$(aws cloudformation --profile preprod --region eu-west-1 describe-stacks --stack-name \"${projectName}\"-\"${username}\" --query Stacks[0].Outputs[0].OutputValue --output text | sed -e 's/^[[:space:]]*//')" = "None" ]
          do
                echo "Webserver deployment not completed.., retrying in 60 sec"
                sleep 60
          done
          echo "http://\$(aws cloudformation --profile preprod --region eu-west-1 describe-stacks --stack-name \"${projectName}\"-\"${username}\" --query Stacks[0].Outputs[0].OutputValue  --output text | sed -e 's/^[[:space:]]*//')"
      """
    }     


     stage("Build Completed Notify Slack"){

        gitCommit = sh(returnStdout: true, script: "git log -1 --pretty=format:'%h %an %s'").trim()
        gitFullSha1 = sh(returnStdout: true, script: "git rev-parse HEAD").trim()

        slackSend message: "${projectName} - build completed. Most recent commit: \n${gitCommit}(<https://github.com/telegraph/${projectName}/commits/${gitFullSha1}|Open>)", token: "${env.SLACK_PLATFORMS_ALERTS}", channel: "${slack_id}", teamDomain: "telegraph", baseUrl: "https://hooks.slack.com/services/", color: "good"
    }
}
