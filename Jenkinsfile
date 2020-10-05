pipeline {
    agent any

    stages {

        stage('Clean') {
            steps {
                gradlew 'clean'
            }
        }

        stage('Integration') {
            parallel {
                stage('Artifacts') {
                    stages {
                        stage('Build App') {
                            steps {
                                gradlew 'classes'
                            }
                        }
                        stage('Unit Test') {
                            steps {
                                gradlew 'allureTest'
                            }
                        }
                        stage('Jacoco Report') {
                            steps {
                                gradlew 'jacocoAllReport'
                            }
                        }
                        stage('Quality Assessment') {
                            steps {
                                timeout(time: 30, unit: 'MINUTES') {
                                    withSonarQubeEnv('Sonarqube') {
                                        gradlew 'sonarqube'
                                    }

                                    waitForQualityGate(
                                        abortPipeline: true,
                                    )
                                }
                            }
                        }
                        stage('Build Artifacts') {
                            steps {
                                gradlew 'archives'
                            }
                        }
                        stage('Javadoc') {
                            steps {
                                gradlew 'allJavadoc'
                                step([
                                    $class:     'JavadocArchiver',
                                    javadocDir: 'build/docs/javadoc',
                                    keepAll:    true
                                ])
                            }
                        }
                    }
                }
            }

            post {
                always {
                    junit([
                        allowEmptyResults: true,
                        keepLongStdio:     true,
                        testResults:       '**/build/test-results/test/TEST-*.xml'
                    ])
                    allure results: [[path: '**/build/allure-results']]
                    jacoco([
                        exclusionPattern: '**/*Spec*',
                        execPattern:      '**/build/jacoco/test.exec'
                    ])
                    archiveArtifacts([
                        artifacts: '**/build/dist/*.tar.gz'
                    ])
                }
            }
        }
    }
}


// Execute Gradle Wrapper
def gradlew(command) {
    sh "./gradlew ${command} --stacktrace"
}


// Execute Maven Wrapper
def mvnw(command) {
    sh "./mvnw ${command}"
}


// Execute `mdpdf` to generate PDF from Markdown
// Current options are follows:
//   - mdpdf
//   - md-to-pdf
def markdown2pdf(path) {
    sh "mdpdf ${path}"
}
