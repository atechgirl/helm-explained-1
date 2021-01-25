pipeline {
    agent { label "chart-testing-agent" }
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage("Adding Dependencies repos") {
            steps{
                container("chart-testing") {
                    sh "helm repo add bitnami https://charts.bitnami.com/bitnami"
                }
            }     
        }

        stage("Lint") {
            steps {
                container("chart-testing") {
                    sh "ct lint --debug"
                }
            }
        }
        stage("Install & Test") {
            steps {
                container("chart-testing") {
                    sh "ct install --upgrade --debug"
                }
            }
        }
        stage("Package Charts") {
            steps {
                script {
                    container("chart-testing") {
                        sh "helm package --dependency-update charts/*"
                    }
                }
            }
        }

        stage("Publish charts to repo") {
            steps {
                script {
                    container("chart-testing") {
                        sh "git clone ${env.GITHUB_PAGES_REPO_URL} chart-repo"
                        def repoType
                        if (env.BRANCH_NAME == "master") {
                            repoType = "stable"
                        } else {
                            repoType = "staging"
                        }

                        def files = sh(script: "ls chart-repo", returnStdout: true)
                        if (!files.contains(repoType)) {
                            sh "mkdir chart-repo/${repoType}"
                        }

                        sh "mv *.tgz chart-repo/${repoType}"
                        
                        sh "helm repo index chart-repo/${repoType}"

                        sh "git config --global user.email 'chartrepo-bot@jenkins.com' "
                        sh "git config --global user.name 'chartrepo-bot'"

                        dir("chart-repo") {
                            sh "git add --all "
                            sh "git commit -m 'pushing charts from branch ${env.BRANCH_NAME}' "
                            sshagent(credentials: ['github-auth-ssh']) {
                                sh("git remote set-url origin 'git@github.com:atechgirl/awesome-charts.git'")
                                sh('git push origin main')
                            }
                        }
                    }
                }
            }
        }
    }
}