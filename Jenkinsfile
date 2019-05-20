pipeline {
    agent {
        node {
            label 'docker-io-ui'
        }
    }
      environment {
        jenkins_openshift_username="c8fdb02e-71cf-4907-963d-19203ee74bb0"
        jenkins_openshift_password="68ec65fb-b20e-4f37-bf26-191bb85d6757"
      }

    options { skipDefaultCheckout() }

    stages {
        stage ('Initialize Application Migration') {
            agent none
            steps {
                script {
                        withCredentials(
                                        [string(credentialsId: jenkins_openshift_username, variable: 'OPENSHIFT_ID'),
                                        string(credentialsId: jenkins_openshift_password, variable: 'OPENSHIFT_PASS')])
                           {
                        

                           
                                stage ('Checkout') {
                                        echo 'Checking out SCM'
                                        checkout scm

                                        sh '''
                                                echo "PATH = ${PATH}"
                                                echo "M2_HOME = ${M2_HOME}"
                                        '''
                                }
                                
                                stage ('Java Build') {
                                        
  
                                             echo 'Ant and Maven Build'
                                             sh 'cd webapp&& mvn clean install'
                                }

                                stage ('Java Tests') {
                                        echo 'Place holder to run Unit tests.'
                                        //sh ''
                                        sh 'sleep 3'

                                }

                                stage ('SonarQube Analysis') {
                                        def gitUrl = sh(returnStdout: true, script: 'git config remote.origin.url').trim()
                                        echo " Performing Sonar Analysis"
                                        sh 'sleep 10'
                                }
                                
                                stage ('Docker Build') {
                                        retry(3) {
                                                sh 'oc login https://openshiftnextgen.in.dst.ibm.com:8443 -u $OPENSHIFT_ID -p $OPENSHIFT_PASS --insecure-skip-tls-verify=true'
                                        }
                                        echo 'creating project for build'
                                        sh 'oc new-project pythonpro-buildcon || oc project pythonpro-buildcon'
                                        echo 'Building and pushing image.'  
                                        sh 'oc delete is pythonpro --ignore-not-found=true && oc delete bc pythonpro --ignore-not-found=true && sleep 5 && oc new-build --binary=true --name pythonpro --allow-missing-imagestream-tags=true && oc start-build --follow --from-dir . pythonpro'
                                        echo 'Pushing image tagged :${env.BUILD_ID}.'
                                }
                                
                                stage ('Openshift deploy') {
                                        echo 'creating project for deployment'
                                        sh 'oc new-project pythonpro-deploy || oc project pythonpro-deploy'
                                        sh 'oc policy add-role-to-user  system:image-puller system:serviceaccount:pythonpro-deploy:default --namespace=pythonpro-buildcon'
                                        echo 'deleting the previous deployment objects if exists'
                                        sh 'oc delete dc pythonpro --ignore-not-found=true'
                                        sh 'oc delete is pythonpro --ignore-not-found=true'
                                        sh 'oc delete svc pythonpro --ignore-not-found=true'
                                        sh 'oc delete route pythonpro --ignore-not-found=true'
                                        sh 'sleep 20'
                                        sh 'oc new-app pythonpro-buildcon/pythonpro --allow-missing-imagestream-tags=true'
                                        sh 'oc delete svc pythonpro --ignore-not-found=true'
                                        echo 'deleting the previous service if exists'
                                        sh 'sleep 3'
                                        sh 'oc expose dc pythonpro --type=LoadBalancer'
                                        sh 'oc expose service/pythonpro'
                                }
                                        
                        }
                }
            }
        }
    }
}
