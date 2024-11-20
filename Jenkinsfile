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
