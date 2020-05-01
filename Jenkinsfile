pipeline{
    agent any
    stages{
        stage('Lint HTML'){
            steps {
                sh '''
					tidy -q -e *.html
					echo "Linting Done"
				'''
            }
        }
        stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'DockerID', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker build -t  myousief/capstone1 .
					'''
				}
			}
		}
        stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'DockerID', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push myousief/capstone1
					'''
				}
			}
		}

		stage('Create kubernetes cluster') {
			steps {
				withAWS(region:'us-east-2', credentials:'AWS') {
					sh '''
						curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
						sudo mv /tmp/eksctl /usr/local/bin
						eksctl version
						eksctl create cluster --name capstone1 --version 1.15 \
												--nodegroup-name standard-workers \
												--node-type t2.small \
												--nodes 2 \
												--nodes-min 1 \
												--nodes-max 3 \
												--node-ami auto \
												--region us-east-2 \
												--zones us-east-2a \
												--zones us-east-2b \
												--zones us-east-2c
					'''
				}
			}
		}

		stage('Create conf file cluster') {
			steps {
				withAWS(region:'us-east-2', credentials:'AWS') {
					sh '''
						aws eks --region us-east-2 update-kubeconfig --name capstone1
					'''
				}
			}
		}

		stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-east-2', credentials:'AWS') {
					sh '''
						kubectl config use-context arn:aws:eks:us-east-2:107490788748:cluster/Capstone1
					'''
				}
			}
		}

		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-east-2', credentials:'AWS') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}

		stage('Deploy green container') {
			steps {
				withAWS(region:'us-east-2', credentials:'AWS') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-east-2', credentials:'AWS') {
					sh '''
						kubectl apply -f ./blue-service.json
					'''
				}
			}
		}

		stage('Wait user approve') {
            steps {
                input "Ready to redirect traffic to green?"
            }
        }

		stage('Create the service in the cluster, redirect to green') {
			steps {
				withAWS(region:'us-east-2', credentials:'AWS') {
					sh '''
						kubectl apply -f ./green-service.json
					'''
				}
			}
		}

		stage('Details of Deployment') {
			steps {
				withAWS(region:'us-east-2', credentials:'AWS') {
					sh '''
						kubectl get pods
					'''
				}
			}
		}

	}
}
