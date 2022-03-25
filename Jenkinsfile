pipeline {
    
    agent any

    stages {
        
        stage('Cloning Git') {
            steps {
                git 'https://github.com/jfcb853/aws-jenkins-pipeline-v4.git'
            }
        }
        stage('Build Docker image & push to ecr') {
            steps {
                sh 'docker build -t script-app .'
                sh 'aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com'
                sh 'docker tag e9ae3c220b23 aws_account_id.dkr.ecr.region.amazonaws.com/my-repository:tag'
                sh 'docker push aws_account_id.dkr.ecr.region.amazonaws.com/my-repository:tag'
            }
        }


        stage('Get the cluster kube config') {
			when {
                branch 'master'
            }
            steps {
				withAWS(credentials: 'aws-credentials', region: 'us-west-2') {
					sh '''
						aws eks --region $region update-kubeconfig --name $cluster_name
						ls ~/.kube/config
                        cat ~/.kube/config
                        kubectl get svc
					'''
				}	
			}
		}

        stage('Deploy to K8s') {
			when {
                branch 'master'
            }
            steps {
				withAWS(credentials: 'aws-credentials', region: 'us-west-2') {
					sh "sed -i 's/latest/${env.BUILD_NUMBER}/g' ./kubernetes/deployment.yaml" 
                    sh '''
                        grep image ./kubernetes/deployment.yaml
                        kubectl apply -f ./kubernetes/deployment.yaml
						sleep 2
                        kubectl get deployments
					'''
				}	
			}
		}

        stage('Service to K8s') {
			when {
                branch 'master'
            }
            steps {
				withAWS(credentials: 'aws-credentials', region: 'us-west-2') {
                    sh '''
                        kubectl get pods -l 'app=express' -o wide | awk {'print $1" " $3 " " $6'} | column -t
                        kubectl get deployments
                        kubectl apply -f ./kubernetes/service.yaml
                        kubectl get services

                    '''
				}	
			}
		}
    }

    post {
        always {
            echo 'Pipeline finished'
        }
    }
}
