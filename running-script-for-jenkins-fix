pipeline {
    agent any
    tools {
        // Use Maven installation configured in Jenkins
        maven 'Maven'
    }

    environment {
        // Define your OpenShift project and application names
        OPENSHIFT_PROJECT = 'demo'
        APP_NAME = 'lab-app'
        IMAGE_NAME = 'lab-app-image'
        REGISTRY = 'registry.access.redhat.com/ubi8/openjdk-17'
        WORKSPACE_DIR = '/var/lib/jenkins/jobs/app-demo4/workspace/target'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/devfile-samples/devfile-sample-java-springboot-basic.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Config') {
            steps {
                script {
                    // Build Config
                    sh "oc login -u admin -p P@ssw0rd https://api.ocp-lab.virtus-demo.com:6443"
                    sh "oc delete project ${OPENSHIFT_PROJECT}"
                    sh "oc new-project ${OPENSHIFT_PROJECT}"
                    sh "oc project ${OPENSHIFT_PROJECT}"
                    sh "oc new-build --name=${APP_NAME} --binary --strategy=docker || echo 'Build config already exists'"
                }
            }
        }

        stage('Build Image') {
            steps {
                script {
                    // Ensure the Dockerfile is present in the workspace directory
                    sh """
                        cd ${WORKSPACE_DIR}
                        echo 'FROM ${REGISTRY}
                        COPY ./demo-0.0.1-SNAPSHOT.jar /deployments/
                        CMD ["java", "-jar", "/deployments/demo-0.0.1-SNAPSHOT.jar"]' > Dockerfile

                        # Ensure the JAR file is present
                        ls -la ${WORKSPACE_DIR}/demo-0.0.1-SNAPSHOT.jar

                        oc start-build ${APP_NAME} --from-dir=${WORKSPACE_DIR} --follow
                    """
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    // Deploy the application using the built image and expose the service
                    sh """
                        oc new-app ${APP_NAME} || echo 'App already exists'
                        oc expose svc/${APP_NAME}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Build successful! Deploying...'
            // Additional post-build actions
        }

        failure {
            echo 'Build failed! Please check the logs.'
            // Additional failure actions
        }
    }
}
