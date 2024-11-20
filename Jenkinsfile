pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *') // poll every 2 mins
    }

    environment {
        JIRA_URL = 'https://jira.default.svc.cluster.local'
        JIRA_USER = 'admin' // for testing purposes, do not use in prod
        JIRA_PASSWORD = 'admin' // for testing purposes, do not use in prod
        JIRA_TRANSITION_ID = '5'
        ISSUE_KEY = ''
    }

    stages {
        stage('Check if merge commit') {
            steps {
                script {
                    // check if last commit is a merge, terminate pipeline if not
                    def hashes = sh(script: "git rev-list --parents -n 1 HEAD", returnStdout: true).trim().split(' ')

                    echo("start${hashes}end")

                    if (hashes.size() < 2) {
                        currentBuild.result = "ABORTED"
                        error("Not a merge commit.")
                    }
                }
            }
        }
        stage('Fetch branch name') {
            steps { 
                script {
                    def branchName = sh(script: 'git branch --contains $(git rev-list --parents -n 1 HEAD | awk "{print \$2}")', ).split('\n')[0]
                    echo("start${branchName}end")
                    env.ISSUE_KEY = branchName
                }
            }
        }
        stage('Set issue to done') {
            steps {
                script {
                    jiraTransitionIssue(issueKey: env.ISSUE_KEY, transitionId: env.JIRA_TRANSITION_ID)
                }
            }
        }
    }
}
