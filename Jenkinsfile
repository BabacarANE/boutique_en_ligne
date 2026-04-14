/**
 * Jenkinsfile – Pipeline CI/CD avec job chaîné
 * Projet : Boutique en ligne – ICDE848
 */

pipeline {

    agent any

    tools {
        maven 'M3'
        jdk 'JDK17'
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

        // ── Stage 1 : Checkout ───────────────────────
        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH}", url: 'https://github.com/BabacarANE/boutique_en_ligne.git'
                echo "Commit : ${env.GIT_COMMIT}"
            }
        }

        // ── Stage 2 : Build ──────────────────────────
        stage('Build') {
            steps {
                sh 'mvn clean package -B -DskipTests'
            }
        }

        // ── Stage 3 : Tests unitaires ────────────────
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
                    echo 'Tests unitaires en ECHEC — vérifier les logs'
                }
            }
        }

        // ── Stage 4 : Couverture JaCoCo ──────────────
        stage('Couverture JaCoCo') {
            steps {
                sh 'mvn jacoco:report -B'
            }
        }

        // ── Stage 5 : Qualité ────────────────────────
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
        }

        // ── Stage 6 : Archive ────────────────────────
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

        // ── Stage 7 : Déclenchement du CD (job chaîné) ──
        stage('Trigger CD Pipeline') {
            steps {
                script {
                    echo "Déclenchement du job CD..."

                    build job: 'cd-boutique',
                    wait: true,
                    propagate: true,
                    parameters: [
                        string(name: 'ENVIRONMENT', value: params.ENVIRONMENT),
                        string(name: 'BRANCH', value: params.BRANCH)
                    ]
                }
            }
        }

    }

    post {

        always {
            echo "Pipeline terminée — statut : ${currentBuild.currentResult}"
        }

        success {
            echo "✅ Build et enchaînement réussis"

            emailext(
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "CI + CD OK : ${env.BUILD_URL}",
                to: 'babacarane58@hotmail.com'
            )
        }

        failure {
            echo "❌ Pipeline en ECHEC"

            emailext(
                subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """\
Projet  : ${env.JOB_NAME}
Build   : #${env.BUILD_NUMBER}
URL     : ${env.BUILD_URL}
Logs    : ${env.BUILD_URL}console
""",
                to: 'babacarane58@hotmail.com',
                attachLog: true
            )
        }

        fixed {
            echo "🔧 Build de nouveau STABLE"

            emailext(
                subject: "✅ FIXED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline est de nouveau stable : ${env.BUILD_URL}",
                to: 'babacarane58@hotmail.com'
            )
        }
    }
}