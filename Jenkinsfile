import groovy.json.JsonSlurperClassic
def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}
pipeline {
    agent any
     environment {
        NEXUS_USER_VAR      = credentials('NEXUS-USER')
        NEXUS_USER_PASS_VAR = credentials('NEXUS-PASS')
        
    }
    stages {
        stage("Paso 1: Compliar"){
            steps {
                script {
                sh "echo 'Compile Code!'"
                // Run Maven on a Unix agent.
                sh "mvn clean compile -e"
                }
            }
        }
        stage("Paso 2: Testear"){
            steps {
                script {
                sh "echo 'Test Code!'"
                // Run Maven on a Unix agent.
                sh "mvn clean test -e"
                }
            }
        }
        stage("Paso 3: Build .Jar"){
            steps {
                script {
                sh "echo 'Build .Jar!'"
                // Run Maven on a Unix agent.
                sh "mvn clean package -e"
                }
            }
            post {
                //record the test results and archive the jar file.
                success {
                    archiveArtifacts artifacts:'build/*.jar'
                }
            }
        }
        stage("Paso 4: Análisis SonarQube"){
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "echo 'Calling sonar Service in another docker container!'"
                    // Run Maven on a Unix agent to execute Sonar.
                    sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=definido-por-mi'
                }
            }
            // post {
            //     //record the test results and archive the jar file.
            //     success {
            //         //archiveArtifacts artifacts:'build/*.jar'
            //         nexusPublisher nexusInstanceId: 'nexus',
            //             nexusRepositoryId: 'devops-usach-nexus',
            //             packages: [
            //                 [$class: 'MavenPackage',
            //                     mavenAssetList: [
            //                         [classifier: '',
            //                         extension: '.jar',
            //                         filePath: 'build/DevOpsUsach2020-0.0.1.jar']
            //                     ],
            //             mavenCoordinate: [
            //                 artifactId: 'DevOpsUsach2020',
            //                 groupId: 'com.devopsusach2020',
            //                 packaging: 'jar',
            //                 version: '0.0.1']
            //             ]
            //         ]
            //     }
            // }
        }
        stage("Paso 5: Levantar Springboot APP"){
            steps {
                sh 'mvn spring-boot:run &'
            }
        }
        stage("Paso 6: Dormir(Esperar 40sg) "){
            steps {
                sh 'sleep 40'
            }
        }
        stage("Paso 7: Test Alive Service - Testing Application!"){
            steps {
                sh 'curl -X GET "http://localhost:8081/rest/mscovid/test?msg=testing"'
            }
        }
        stage('Paso 8:Subir Nexus') {
            steps {
                echo 'Subiendo a nexus desde código'
                nexusPublisher nexusInstanceId: 'nexus', nexusRepositoryId: 'devops-usach-nexus', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: '/var/jenkins_home/workspace/job-github-sonar-3/build/DevOpsUsach2020-0.0.1.jar']], mavenCoordinate: [artifactId: 'DevOpsUsach2020', groupId: 'com.devopsusach2020', packaging: 'jar', version: '0.0.1-AS-CODE']]]
            }
        }
        stage('Paso 9:Bajar Nexus') {
            steps {
                echo 'Bajando a nexus desde código'
                sh 'curl -X GET -u $NEXUS_USER_VAR:$NEXUS_USER_PASS_VAR http://nexus:8081/repository/devops-usach-nexus/com/devopsusach2020/DevOpsUsach2020/0.0.1-AS-CODE/DevOpsUsach2020-0.0.1-AS-CODE.jar -O'
            }
        }
    stage("Paso 10: Levantar Springboot APP"){
            steps {
                sh 'mvn spring-boot:run &'
                sh 'nohup bash java -jar DevOpsUsach2020-0.0.1.jar & >/dev/null'
            }
        }
        stage("Paso 11: Dormir(Esperar 40sg) "){
            steps {
               sh "sleep 40 && curl -X GET 'http://localhost:8081/rest/mscovid/test?msg=testing'"
            }
        }
        stage("Paso 12: Subir nueva Version"){
            steps {
                //archiveArtifacts artifacts:'build/*.jar'
                nexusPublisher nexusInstanceId: 'nexus',
                    nexusRepositoryId: 'devops-usach-nexus',
                    packages: [
                        [$class: 'MavenPackage',
                            mavenAssetList: [
                                [classifier: '',
                                extension: 'jar',
                                filePath: 'build/DevOpsUsach2020-0.0.1.jar']
                            ],
                    mavenCoordinate: [
                        artifactId: 'DevOpsUsach2020',
                        groupId: 'com.devopsusach2020',
                        packaging: 'jar',
                        version: '0.0.4']
                    ]
                ]
            }
        }
    }
    post {
        always {
            sh "echo 'fase always executed post'"
        }
        success {
            sh "echo 'fase success'"
        }
        failure {
            sh "echo 'fase failure'"
        }
    }
}
