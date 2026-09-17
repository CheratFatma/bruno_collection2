pipeline {
    agent any

    parameters {
        booleanParam(
            name: 'ALLURE',
            defaultValue: false,
            description: 'Génération du rapport Allure'
        )
    }

    stages {

        stage('Global stage') {
            agent {
                docker {
                    image 'node:latest'
                    args '-u root'
                }
            }

            stages {

                stage('Installer Bruno') {
                    steps {
                        sh '''
                            echo "Installation de Bruno CLI..."
                            npm install -g @usebruno/cli
                            bru --version
                        '''
                    }
                }

                stage('Installer Allure') {
                    steps {
                        sh '''
                            echo "Installation du reporter Allure..."
                            npm install -g allure-commandline
                        '''
                    }
                }

                stage('Clean Allure results') {
                    steps {
                        sh '''
                            echo "Suppression des anciens résultats Allure..."
                            rm -rf allure-results
                            mkdir -p allure-results
                            echo "Dossier allure-results nettoyé avec succès"
                        '''
                    }
                }

                stage('Run user test') {
                    steps {
                        script {

                            if (params.ALLURE) {

                                sh '''
                                    bru run --env-file ./environments/preprod.yml --reporter-json results.json --reporter-junit results.xml
                                '''

                                stash(
                                    name: 'allure-results',
                                    includes: 'allure-results/*',
                                    allowEmpty: true
                                )

                            } else {

                                sh '''
                                    echo "Exécution des tests Bruno..."

                                    bru run --env-file ./environments/preprod.yml
                                '''
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            script {

                if (params.ALLURE) {

                    unstash 'allure-results'

                    archiveArtifacts(
                        artifacts: 'allure-results/*',
                        allowEmptyArchive: true
                    )

                    allure(
                        includeProperties: false,
                        jdk: '',
                        results: [[path: 'allure-results/']]
                    )
                }
            }
        }
    }
}