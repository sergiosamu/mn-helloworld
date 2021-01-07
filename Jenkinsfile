properties([
    parameters([                                       
        gitParameter(branch: '',
                     branchFilter: 'origin/(.*)',
                     defaultValue: 'master',
                     description: '',
                     name: 'BRANCH',
                     quickFilterEnabled: false,
                     selectedValue: 'NONE',
                     sortMode: 'NONE',
                     tagFilter: '*',
                     type: 'PT_BRANCH')
    ])
])

pipeline {
   agent any

   stages {
      stage('Build application') {
         steps {
            withMaven(maven: 'Maven 3.5.0', jdk: 'JDK8') {
                sh "mvn versions:set -DnewVersion=$BRANCH"
                sh "mvn clean package -U"
            }
         }
      }
      stage('Build docker image') {
         steps {
            sh "docker build . -t helloworld:$BRANCH"
            sh "docker tag helloworld:$BRANCH 757992231822.dkr.ecr.eu-west-1.amazonaws.com/helloworld:$BRANCH"
         }
      }   
      stage('Push image to AWS') {
         steps {
            withDockerRegistry(credentialsId: 'ecr:eu-west-1:ecr_user', url: 'https://757992231822.dkr.ecr.eu-west-1.amazonaws.com') {
                sh "docker push 757992231822.dkr.ecr.eu-west-1.amazonaws.com/helloworld:$BRANCH"
            }
         }
      } 
      stage('Deploy to k8s') {
         steps {
            ansiblePlaybook(installation: 'Ansible 2.8', 
                            playbook: 'ansible-deploy.yml',
                            extraVars: [ 
                               version: "$BRANCH" ]
                            );
         }
      }       
   }
}
