pipeline {
  environment {
    registry = '120700353163.dkr.ecr.us-east-2.amazonaws.com/hello-world'
    registryCredential = 'ecr-creds'
    AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
    AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    dockerImage = ''
  }
  agent any

  stages {
    stage ('Build') {
		   steps {
		      sh '''
			  cd $WORKSPACE
                          chmod +x mvnw
			   ./mvnw clean install package
                           cp $WORKSPACE/webapp/target/webapp.war $WORKSPACE
			  '''
			  }
      }
    stage('Building image') {
      steps{
	      sh '''
	      docker build -t ${registry}:${BUILD_NUMBER} .
	      '''
      }
    }
    stage('Deploy image') {
        steps{
	   sh '''
	     aws ecr get-login --region us-east-2 --no-include-email | xargs -n 1 -P 10 -I {} bash -c {}
	      docker push ${registry}:${BUILD_NUMBER}
	    '''
        }
    }
    stage ('Deploy to k8s') {
		   steps {
		      sh '''
			  cd $WORKSPACE

			   aws eks update-kubeconfig --name test-dev-eks --region us-east-1
                           /usr/local/bin/helm upgrade --install hello-world helm/hello-world --set image.tag="${BUILD_NUMBER}"
			  '''
			      }
            }
  }
}
