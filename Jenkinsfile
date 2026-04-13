/**
 * Jenkinsfile – Pipeline CI complète
 * Projet : Boutique en ligne – ICDE848
 */

pipeline {

    agent any

    tools {
        maven 'M3'
        jdk 'JDK17' // Décommenter après avoir configuré JDK dans Global Tool Configuration
    }

    parameters {
        string(
            name: 'BRANCH',
            defaultValue: 'main',
            description: 'Branche Git à builder'
        )

        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'staging', 'prod'],
            description: 'Environnement de déploiement cible'
        )

        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Ignorer les tests (urgence uniquement !)'
        )
    }

    stages {

        // ── Stage 1 : Récupérer le code ──────────────
        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH}", url: 'https://github.com/BabacarANE/boutique_en_ligne.git'
                echo "Commit : ${env.GIT_COMMIT}"
            }
        }

        // ── Stage 2 : Compiler ───────────────────────
        stage('Build') {
            steps {
                sh 'mvn clean compile -B'
            }
        }

        // ── Stage 3 : Tests unitaires ─────────────────
        stage('Tests unitaires') {
            when {
                not { expression { params.SKIP_TESTS } }
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

        // ── Stage 4 : Tests d'intégration ────────────
        /*
        stage('Tests intégration') {
            when {
                not { expression { params.SKIP_TESTS } }
            }
            steps {
                sh 'mvn verify -Dsurefire.skip=true -B'
            }
            post {
                always {
                    junit '**/target/failsafe-reports/*.xml'
                }
            }
        }
        */

        // ── Stage 5 : Couverture de code ─────────────
        stage('Couverture JaCoCo') {
            steps {
                sh 'mvn jacoco:report -B'
                // Rapport généré ici : target/site/jacoco/index.html
            }

            /*
            post {
                always {
                    jacoco(
                        execPattern: '**/target/jacoco.exec',
                        classPattern: '**/target/classes',
                        sourcePattern: '**/src/main/java',
                        minimumLineCoverage: '70'
                    )
                }
            }
            */
        }

        // ── Stage 6 : Analyse qualité ─────────────────
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

            /*
            post {
                always {
                    recordIssues(
                        enabledForFailure: true,
                        tools: [
                            checkStyle(pattern: '**/checkstyle-result.xml'),
                            pmdParser(pattern: '**/pmd.xml'),
                            cpd(pattern: '**/cpd.xml'),
                            spotBugs(pattern: '**/spotbugsXml.xml')
                        ],
                        qualityGates: [[
                            threshold: 10,
                            type: 'TOTAL',
                            unstable: true
                        ]]
                    )
                }
            }
            */
        }

        // ── Stage 7 : Archiver le JAR ─────────────────
        stage('Archive') {
            steps {
                archiveArtifacts(
                    artifacts: '**/target/*.jar',
                    fingerprint: true,
                    allowEmptyArchive: false
                )
                echo "Artefact archivé avec succès"
            }
        }

        // ── Stage 8 : Validation PROD ─────────────────
        /*
        stage('Validation PROD') {
            when {
                expression { params.ENVIRONMENT == 'prod' }
            }
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input(
                        message: "Déployer en PRODUCTION ?",
                        ok: "Oui, déployer",
                        submitter: "admin,tech-lead"
                    )
                }
            }
        }
        */

        // ── Stage 9 : Déploiement ─────────────────────
        /*
        stage('Deploy') {
            steps {
                sh "./deploy.sh ${params.ENVIRONMENT}"
            }
        }
        */

    }

    post {

        always {
            echo "Pipeline terminée — statut : ${currentBuild.currentResult}"
        }

        failure {
            echo "❌ Build en ECHEC — vérifier les logs"
        }

        success {
            echo "✅ Build réussi"
        }

        /*
        failure {
            emailext(
                subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Projet  : ${env.JOB_NAME}
Build   : #${env.BUILD_NUMBER}
URL     : ${env.BUILD_URL}
                """,
                to: 'ton-email@gmail.com',
                attachLog: true
            )
        }

        fixed {
            emailext(
                subject: "✅ FIXED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build restauré : ${env.BUILD_URL}",
                to: 'ton-email@gmail.com'
            )
        }
        */
    }
}