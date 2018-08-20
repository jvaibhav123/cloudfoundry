#!/usr/bin/env groovy
pipeline {
  
environment{
	CF_DOCKER_PASSWORD="$dockerhubpwd"
}
agent any 


  stages {
 
	
    stage('checkout') {
		steps{
		script {
		git credentialsId: 'e0c038d8-5106-4d22-87e5-16b018816ef7', url: 'https://github.com/jvaibhav123/cloudfoundry.git'
		env.BUILD_TAG=v13
		}
	   }	
	}
	
	
	stage('Build Plus Push')
	{
	  when{
		expression {
				return "$BUILD_TAG"!=""
		}
	}
	  steps{
	  
	  script {
		withCredentials([usernamePassword(credentialsId: 'dockkerhub', passwordVariable: 'DOCKERPWD', usernameVariable: 'DOCKERUSER')]) {
		echo "Build tag ${BUILD_TAG}"
		if ("${BUILD_TAG}" != ""){
		stage('Build Docker image')
		{
			sh """
		
			echo "Docker build image"
			docker build -t $DOCKERUSER/demoapp:${BUILD_TAG} .
			
			
			"""
			
		}
			
		 stage('Test the image'){
		  when{
                	expression {
                                return "$BUILD_TAG"!=""
                }
        }

		   def testImageExist
                   sh """
		   testImageExist=`docker ps --filter "name=test_image" -q | wc -l`
		   if [ \$testImageExist -ge 1 ]
		   then
			docker stop \$(docker ps --filter "name=test_image" -q )
			docker rm \$(docker ps -a --filter "name=test_image" -q )
		   fi

		   testImageExist=`docker ps -a --filter "name=test_image" -q | wc -l`
		   if [ \$testImageExist -ge 1 ]
                   then
                        docker rm \$(docker ps -a --filter "name=test_image" -q )
                   fi

                   echo "Run container with latest image"
                   docker run -itd --name test_image -p 6000:5000 -l app=testimage $DOCKERUSER/demoapp:${BUILD_TAG}
                   echo "docker container successfully  started"
                   sleep 5
                   if [ `curl -s -o /dev/null -w "%{http_code}\n" http://0.0.0.0:6000/` -eq 200 ]
                   then
                                        echo "Docker image successfully running"
                   else
                                        echo "Something wrong"

                   fi

                   docker stop \$(docker ps --filter "name=test_image" -q )
                   docker rm \$(docker ps -a --filter "name=test_image" -q )
                   """


                 }
	

		 stage('Push to docker hub'){
		  when{
                expression {
                                return "$BUILD_TAG"!=""
                }
        }

		 sh """
		docker login --username $DOCKERUSER --password $DOCKERPWD
		echo "Docker login successful"
		 echo "Push docker image"
		 docker push $DOCKERUSER/demoapp:${BUILD_TAG}
		 echo "Push completed successfully"
		 """
		 }
		 
		 
		 stage('Deploy the image'){
		  when{
                expression {
                                return "$BUILD_TAG"!=""
                  }
        	}

		 pushToCloudFoundry cloudSpace: 'pcfdev-space', credentialsId: 'cloudfoundry', organization: 'pcfdev-org', selfSigned: 'true', target: 'api.local.pcfdev.io'
		 sh """

		 cf push --docker-image docker200037/demoapp:$BUILD_TAG 
		 """		 
 
		 }
		  
			
		}
		}

	}
	
	}
	
	}
	
}



}




