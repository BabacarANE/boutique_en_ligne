/**
 * Jenkinsfile – Pipeline CI complète
 * Projet : Boutique en ligne – ICDE848
 *
 * Ce fichier doit être placé à la RACINE du dépôt Git.
 * Jenkins le détecte automatiquement lors de la création du job Pipeline.
 *
 * Stages :
 *   1. Checkout       → récupère le code depuis Git
 *   2. Build          → compile le code source
 *   3. Tests unitaires → lance *Test.java via Surefire
 *   4. Tests intégration → lance *IT.java via Failsafe
 *   5. Couverture     → génère le rapport JaCoCo
 *   6. Qualité        → Checkstyle + PMD + CPD + SpotBugs
 *   7. Archive        → sauvegarde le JAR dans Jenkins
 *
 * Post :
 *   - failure → email à l'équipe
 *   - fixed   → email quand le build repasse au vert
 */

pipeline {

    agent any

    tools {
        maven 'M3'
    }

    parameters {
        string(
            name:         'BRANCH',
            defaultValue: 'main',
            description:  'Branche Git à builder'
        )
        choice(
            name:    'ENVIRONMENT',
            choices: ['dev', 'staging', 'prod'],
            description: 'Environnement de déploiement cible'
        )
        booleanParam(
            name:         'SKIP_TESTS',
            defaultValue: false,
            description:  'Ignorer les tests (urgence uniquement !)'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/BabacarANE/boutique_en_ligne.git'
                echo "Commit : ${env.GIT_COMMIT}"
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile -B'
            }
        }

        stage('Tests unitaires') {
            when {
                not { expression { return params.SKIP_TESTS } }
            }
            steps {
                sh 'mvn test -B'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
                failure {
                    echo 'Tests unitaires en ECHEC — vérifier les logs ci-dessus'
                }
            }
        }

        stage('Couverture JaCoCo') {
            steps {
                sh 'mvn jacoco:report -B'
            }
            post {
                always {
                    jacoco(
                        execPattern:   '**/target/jacoco.exec',
                        classPattern:  '**/target/classes',
                        sourcePattern: '**/src/main/java'
                    )
                }
            }
        }

        stage('Qualité') {
            steps {
                sh '''
                    mvn checkstyle:checkstyle \
                        pmd:pmd \
                        pmd:cpd \
                        spotbugs:spotbugs \
                        -B
                '''
            }
            post {
                always {
                    recordIssues(
                        enabledForFailure: true,
                        tools: [
                            checkStyle(pattern: '**/checkstyle-result.xml'),
                            pmdParser(pattern:  '**/pmd.xml'),
                            cpd(pattern:        '**/cpd.xml'),
                            spotBugs(pattern:   '**/spotbugsXml.xml')
                        ],
                        qualityGates: [[
                            threshold: 10,
                            type: 'TOTAL',
                            unstable: true
                        ]]
                    )
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts(
                    artifacts:         '**/target/*.jar',
                    fingerprint:       true,
                    allowEmptyArchive: false
                )
                echo "Artefact archivé avec succès"
            }
        }

    }

    post {
        always {
            echo "Pipeline terminée — statut : ${currentBuild.currentResult}"
        }
        failure {
            emailext(
                subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Le build a échoué.

Projet  : ${env.JOB_NAME}
Build   : #${env.BUILD_NUMBER}
URL     : ${env.BUILD_URL}

Consulter les logs : ${env.BUILD_URL}console
                """,
                to:        'ton-email@gmail.com',
                attachLog: true
            )
        }
        fixed {
            emailext(
                subject: "✅ FIXED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body:    "Le build est de nouveau stable : ${env.BUILD_URL}",
                to:      'anebabacar221@gmail.com'
            )
        }
    }

}