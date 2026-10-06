pipeline {

    agent any

    // ============================================================
    // ENVIRONMENT VARIABLES
    // ============================================================

    environment {

        // DockerHub
        DOCKERHUB_USERNAME = 'adeem07'
        IMAGE_NAME = 'shopping-cart'
    }


    tools {

        // Java 8 : utilisé pour compiler le projet Spring Boot legacy
        jdk 'java8'

        // Maven configuré dans Jenkins
        maven 'maven3'
    }


    stages {

        // ============================================================
        // 1. CLONE
        // ============================================================

        stage('Clone Repository') {

            steps {

                checkout([
                    $class: 'GitSCM',

                    branches: [[
                        name: '*/main'
                    ]],

                    userRemoteConfigs: [[
                        credentialsId:
                            'c27165f1-27db-49f1-86ce-fe0e1aa6e0fb',

                        url:
                            'https://github.com/AdemZe/Jenkins-Lab.git'
                    ]]
                ])

                sh '''
                    echo "========================================"
                    echo "       REPOSITORY INFORMATION"
                    echo "========================================"

                    git branch --show-current
                    git log -1 --oneline

                    echo ""

                    echo "Repository cloned successfully."
                '''
            }
        }


        // ============================================================
        // 2. CHECK JAVA 8
        // ============================================================

        stage('Check Java 8') {

            steps {

                sh '''
                    echo "========================================"
                    echo "           JAVA 8 INFORMATION"
                    echo "========================================"

                    echo "JAVA_HOME=$JAVA_HOME"

                    java -version

                    echo ""

                    echo "========================================"
                    echo "           MAVEN WRAPPER"
                    echo "========================================"

                    ./scripts/mvnw -version
                '''
            }
        }


        // ============================================================
        // 3. MAVEN PACKAGE
        // ============================================================

        stage('Maven Package') {

            steps {

                sh '''
                    echo "========================================"
                    echo "           MAVEN PACKAGE"
                    echo "========================================"

                    ./scripts/mvnw clean package

                    echo ""

                    echo "========================================"
                    echo "           BUILD ARTIFACT"
                    echo "========================================"

                    ls -lh target/

                    echo ""

                    echo "JAR files:"
                    ls -lh target/*.jar
                '''
            }
        }


        // ============================================================
        // 4. SONARQUBE ANALYSIS
        // ============================================================

        stage('SonarQube Analysis') {

            steps {

                script {

                    def java21 =
                        tool(
                            name: 'java21',
                            type: 'jdk'
                        )


                    withEnv([
                        "JAVA_HOME=${java21}",
                        "PATH+JAVA=${java21}/bin"
                    ]) {

                        withSonarQubeEnv('SonarQube') {

                            sh '''
                                echo "========================================"
                                echo "        SONARQUBE ANALYSIS"
                                echo "========================================"

                                echo "JAVA_HOME=$JAVA_HOME"

                                java -version

                                echo ""

                                echo "SONAR_HOST_URL=$SONAR_HOST_URL"

                                echo ""

                                echo "Starting SonarQube analysis..."

                                ./scripts/mvnw \
                                  org.sonarsource.scanner.maven:sonar-maven-plugin:5.8.0.7211:sonar \
                                  -Dsonar.host.url="$SONAR_HOST_URL"
                            '''
                        }
                    }
                }
            }
        }


        // ============================================================
        // 5. SONARQUBE QUALITY GATE
        // ============================================================

        stage('SonarQube Quality Gate') {

            steps {

                echo "Waiting for SonarQube Quality Gate..."

                timeout(
                    time: 10,
                    unit: 'MINUTES'
                ) {

                    waitForQualityGate(
                        abortPipeline: true
                    )
                }
            }
        }


        // ============================================================
        // 6. OWASP DEPENDENCY CHECK
        // ============================================================

        stage('OWASP Dependency Check') {

            steps {

                script {

                    def java21 =
                        tool(
                            name: 'java21',
                            type: 'jdk'
                        )


                    withEnv([
                        "JAVA_HOME=${java21}",
                        "PATH+JAVA=${java21}/bin"
                    ]) {

                        withCredentials([
                            string(
                                credentialsId: 'nvd-api-key',
                                variable: 'NVD_API_KEY'
                            )
                        ]) {

                            sh '''
                                echo "========================================"
                                echo "       JAVA FOR OWASP"
                                echo "========================================"

                                echo "JAVA_HOME=$JAVA_HOME"

                                java -version

                                echo ""

                                echo "========================================"
                                echo "       CREATE REPORT DIRECTORY"
                                echo "========================================"

                                mkdir -p dependency-check-report

                                echo ""

                                echo "========================================"
                                echo "       OWASP DEPENDENCY CHECK"
                                echo "========================================"
                            '''


                            dependencyCheck(

                                additionalArguments: """
                                    --scan .
                                    --format HTML
                                    --format XML
                                    --format JSON
                                    --out dependency-check-report
                                    --nvdApiKey ${NVD_API_KEY}
                                """,

                                odcInstallation:
                                    'DP'
                            )
                        }
                    }
                }
            }
        }


        // ============================================================
        // 7. PUBLISH OWASP REPORT
        // ============================================================

        stage('Publish OWASP Report') {

            steps {

                dependencyCheckPublisher(
                    pattern:
                        'dependency-check-report/dependency-check-report.xml'
                )
            }
        }


        // ============================================================
        // 8. DOCKER BUILD
        // ============================================================

        stage('Docker Build') {

            steps {

                script {

                    echo '''
========================================
           DOCKER BUILD
========================================
'''

                    def VERSION =
                        "v${env.BUILD_NUMBER}"

                    def IMAGE =
                        "${env.DOCKERHUB_USERNAME}/${env.IMAGE_NAME}"


                    echo "DockerHub Username : ${env.DOCKERHUB_USERNAME}"
                    echo "Image              : ${IMAGE}"
                    echo "Version            : ${VERSION}"


                    sh """
                        echo "========================================"
                        echo "          DOCKER INFORMATION"
                        echo "========================================"

                        docker --version

                        echo ""

                        echo "========================================"
                        echo "          DOCKER BUILD"
                        echo "========================================"

                        docker build \\
                            -f docker/Dockerfile \\
                            -t ${IMAGE}:${VERSION} \\
                            -t ${IMAGE}:latest \\
                            .
                    """
                }
            }
        }


        // ============================================================
        // 9. TRIVY IMAGE SCAN
        // ============================================================

        stage('Trivy Image Scan') {

            steps {

                script {

                    def VERSION =
                        "v${env.BUILD_NUMBER}"

                    def IMAGE =
                        "${env.DOCKERHUB_USERNAME}/${env.IMAGE_NAME}:${VERSION}"


                    echo """
========================================
          TRIVY IMAGE SCAN
========================================

Image : ${IMAGE}

IMPORTANT:
Trivy peut trouver des vulnérabilités HIGH/CRITICAL.
Dans ce lab éducatif, le scan ne bloque PAS le pipeline.
"""


                    catchError(
                        buildResult: 'UNSTABLE',
                        stageResult: 'UNSTABLE'
                    ) {

                        sh """
                            echo "========================================"
                            echo "          TRIVY INFORMATION"
                            echo "========================================"

                            trivy --version

                            echo ""

                            echo "========================================"
                            echo "          STARTING TRIVY SCAN"
                            echo "========================================"

                            trivy image \\
                                --timeout 15m \\
                                --scanners vuln \\
                                --severity HIGH,CRITICAL \\
                                --exit-code 1 \\
                                ${IMAGE}

                            echo ""

                            echo "========================================"
                            echo "          TRIVY SCAN FINISHED"
                            echo "========================================"
                        """
                    }

                    echo """
========================================
       TRIVY SCAN COMPLETED
========================================

Pipeline continues even if
HIGH/CRITICAL vulnerabilities were found.

This is intentional for the educational lab.
"""
                }
            }
        }


        // ============================================================
        // 10. SNYK CONTAINER SCAN
        // ============================================================

        stage('Snyk Container Scan') {

            steps {

                script {

                    def VERSION =
                        "v${env.BUILD_NUMBER}"

                    def IMAGE =
                        "${env.DOCKERHUB_USERNAME}/${env.IMAGE_NAME}:${VERSION}"


                    echo """
========================================
          SNYK CONTAINER SCAN
========================================

Image : ${IMAGE}

IMPORTANT:
Snyk peut trouver des vulnérabilités.
Dans ce lab éducatif, le scan ne bloque PAS le pipeline.
"""


                    catchError(
                        buildResult: 'UNSTABLE',
                        stageResult: 'UNSTABLE'
                    ) {

                        withCredentials([
                            string(
                                credentialsId: 'snyk-token',
                                variable: 'SNYK_TOKEN'
                            )
                        ]) {

                            withEnv([
                                "IMAGE_TO_SCAN=${IMAGE}"
                            ]) {

                                sh '''
                                    echo "========================================"
                                    echo "          SNYK INFORMATION"
                                    echo "========================================"

                                    snyk --version

                                    echo ""

                                    echo "========================================"
                                    echo "          SNYK AUTHENTICATION"
                                    echo "========================================"

                                    snyk auth "$SNYK_TOKEN"

                                    echo ""

                                    echo "========================================"
                                    echo "          STARTING SNYK SCAN"
                                    echo "========================================"

                                    snyk container test "$IMAGE_TO_SCAN"

                                    echo ""

                                    echo "========================================"
                                    echo "          SNYK SCAN FINISHED"
                                    echo "========================================"
                                '''
                            }
                        }
                    }

                    echo """
========================================
       SNYK SCAN COMPLETED
========================================

Pipeline continues even if
Snyk reports vulnerabilities.

This is intentional for the educational lab.
"""
                }
            }
        }


        // ============================================================
        // 11. DOCKER PUSH
        // ============================================================

        stage('Docker Push') {

            steps {

                script {

                    def VERSION =
                        "v${env.BUILD_NUMBER}"

                    def IMAGE =
                        "${env.DOCKERHUB_USERNAME}/${env.IMAGE_NAME}"


                    echo """
========================================
          DOCKER PUSH
========================================

Image  : ${IMAGE}
Version: ${VERSION}
"""


                    withDockerRegistry(
                        credentialsId:
                            '941b6432-e9a7-4a7a-a4cf-44434bd6f00b'
                    ) {

                        sh """
                            echo "========================================"
                            echo "          DOCKER PUSH"
                            echo "========================================"

                            docker push ${IMAGE}:${VERSION}

                            docker push ${IMAGE}:latest

                            echo ""

                            echo "========================================"
                            echo "          PUSH SUCCESSFUL"
                            echo "========================================"

                            echo "Version pushed : ${IMAGE}:${VERSION}"
                            echo "Latest pushed  : ${IMAGE}:latest"
                        """
                    }
                }
            }
        }
    }


    // ================================================================
    // POST ACTIONS
    // ================================================================

    post {

        success {

            echo '''
========================================
       PIPELINE SUCCESS
========================================

GitHub
   ↓
Checkout
   ↓
Java 8
   ↓
Maven Package
   ↓
SonarQube Analysis
   ↓
Quality Gate
   ↓
OWASP Dependency Check
   ↓
OWASP Report
   ↓
Docker Build
   ↓
Trivy Image Scan
   ↓
Snyk Container Scan
   ↓
DockerHub Push

ALL COMPLETED SUCCESSFULLY
========================================
'''
        }


        failure {

            echo '''
========================================
       PIPELINE FAILED
========================================

Check Jenkins Console Output.

Possible areas:

- GitHub credentials
- Git clone
- Java 8
- Maven
- Compilation
- Java 21
- SonarQube
- SonarQube token
- SonarQube Quality Gate
- SonarQube webhook
- OWASP Dependency Check
- NVD API
- OWASP report
- Docker
- Docker build
- Trivy
- Snyk
- Snyk credentials
- DockerHub credentials
- Docker push

========================================
'''
        }


        always {

            echo '''
========================================
       PIPELINE FINISHED
========================================
'''
        }
    }
}