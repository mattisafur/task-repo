pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *') // poll every 2 mins
    }

    environment {
        JIRA_URL = 'https://jira.default.svc.cluster.local'
        JIRA_USER = 'admin' // for testing purposes, do not use in prod
        JIRA_PASSWORD = 'admin' // for testing purposes, do not use in prod
        JIRA_KEY = ''
    }


    stages {
        stage('Check if merge commit') {
            steps {
                script {
                    // Check if the current commit is a merge commit
                    def isMergeCommit = sh(script: 'git log -1 --pretty=%P', returnStdout: true).trim()
                    
                    // If it's a merge commit, there will be two parents, so the output will have two commit hashes
                    if (isMergeCommit.split().size() == 2) {
                        echo "This is a merge commit. Proceeding with further steps."
                    } else {
                        echo "This is not a merge commit. Skipping the job."
                        currentBuild.result = 'SUCCESS'
                        return // Exit the pipeline as this is not a merge commit
                    }
                }
            }
       }
        stage('Fetch branch name') {
            steps {
                script {
                    env.JIRA_KEY = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    echo "JIRA_KEY: ${JIRA_KEY}"
                }
            }
        }
        stage('Mark as done') {
            steps {
                script {
                    def jiraUrl = "${JIRA_URL}/rest/api/2/issue/${JIRA_KEY}/transitions"
                    // set "id" to the ID of the Done state
                    def jsonBody = '''
                    {
                        "transition": {
                            "id": "5"
                        }
                    }
                    '''

                    sh """
                        curl -u ${JIRA_USER}:${JIRA_PASSWORD} -X POST ${jiraUrl} -H 'Content-Type: application/json' -d '${jsonBody}'
                    """
                }
            }
        }
    }
}
