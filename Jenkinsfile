pipeline {
    agent {
        label 'maven'
    }

    stages {
        stage('Build App') {
            steps {
                git branch: 'main', url: 'https://github.com/kuldeepsingh99/openshift-jenkins-cicd.git'
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    version = pom.version
                }
                sh "mvn install"
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-app:latest .'
                sh 'docker tag my-app:latest my-registry/my-app:latest'
                sh 'docker push my-registry/my-app:latest'
            }
        }
        stage('Deploy to OpenShift') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            def dc = openshift.selector('dc', 'my-app')
                            if (dc.exists()) {
                                dc.rollout().latest()
                            } else {
                                openshift.newApp('my-registry/my-app:latest', '--name=my-app')
                                openshift.selector('svc', 'my-app').expose()
                            }
                        }
                    }
                }
            }
        }
    }
}
