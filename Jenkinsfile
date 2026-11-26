pipeline {
    agent { label 'wsl' }

    triggers {
        pollSCM('H/5 * * * *')
    }

    parameters {
        booleanParam(name: 'PUBLISH', defaultValue: false,
                     description: 'Rulează operația cu credential (doar pe master)')
    }

    options {
        disableConcurrentBuilds()
    }

    stages {
        stage('Compile') {
            steps {
                // Rapoartele vechi nu trebuie publicate pentru buildul curent
                sh 'rm -rf backend/build/test-results'
                dir('backend') {
                    sh './gradlew --no-daemon clean classes'
                }
            }
        }

        stage('Test') {
            steps {
                dir('backend') {
                    sh './gradlew --no-daemon test'
                }
            }
        }

        stage('Operație cu credential') {
            when {
                allOf {
                    expression { env.GIT_BRANCH == 'origin/master' }
                    expression { params.PUBLISH }
                }
            }
            steps {
                withCredentials([string(credentialsId: 'lab-github-token', variable: 'GH_TOKEN')]) {
                    sh '''
                        set +x
                        curl -fsS -o /dev/null -w "GitHub API: %{http_code}\\n" \
                          -H "Authorization: Bearer $GH_TOKEN" \
                          https://api.github.com/repos/RBROL02083/task-tracker
                    '''
                }
            }
        }
    }

    post {
        always {
            junit testResults: 'backend/build/test-results/test/TEST-*.xml', allowEmptyResults: false
        }
    }
}
