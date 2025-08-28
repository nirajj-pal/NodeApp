pipeline {
	agent any
	tools {
		nodejs 'NodeJS'
	}
	environment {
		DOCKER_HUB_CREDENTIALS_ID = 'jen-dockerhub'
		DOCKER_HUB_REPO = 'iquantc/iquant-app'
	}
	stages {
		stage('Checkout Github'){
			steps {
				git branch: 'main', credentialsId: 'jen-doc-git', url: 'https://github.com/nirajj-pal/NodeApp.git'
			}
		}		
		stage('Install node dependencies'){
			steps {
				sh 'npm install'
			}
		}
		stage('Test Code'){
			steps {
				sh 'npm test'
			}
		}
		stage('Build Docker Image'){
			steps {
				script {
					dockerImage = docker.build("${DOCKER_HUB_REPO}:latest")
				}
			}
		}
		stage('Trivy Scan'){
			steps {
				sh 'trivy --severity HIGH,CRITICAL --no-progress image --format table -o trivy-scan-report.txt ${DOCKER_HUB_REPO}:latest'
			}
		}
		stage('Push Image to DockerHub'){
			steps {
				script {
					docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_HUB_CREDENTIALS_ID}"){
						dockerImage.push('latest')
					}
				}
			}
		}
		stage('Install Kubectl'){
			steps {
				sh '''
				curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                		chmod +x kubectl
                		mv kubectl /usr/local/bin/kubectl
				'''
			}
		}
		stage('Deploy to Kubernetes'){
			steps {
				script {
					kubeconfig(caCertificate: 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJZEl3VGJVcCtKRHN3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TlRBNE1qVXhNVEUwTWpkYUZ3MHpOVEE0TWpNeE1URTVNamRhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUURqc0QxUVphbjZaMUlxZVNYNlYwcWp0MUN5ZkpTaUVPcGpvUzl5QjYvVGdYeW8xQ1lTMVUzLytrSWUKenZkbzVRUjNKSDJKaW1CblFIUXVUbWQzb0lTdElnUGdZbzVpNU5uL2VtOURvcjd0ZTcrc0xaVXZ3eEU4YkNJZwo5bUxTMml2TWhndC84ajlrVlAzNEFFdE9ua1c4aW9Da0VaSXVLL01xeklNVlVsWXMzTU5EbVVvQmhmQm42enh5CnRjUDRuRU9kOFp6YUtvMWRJcHExN0RoRWZEbU5HVGZSTWp5QUs4OEtWVzF1Slg1UXVBQTJiSTBEeVVkZzhvdmEKWFQ0TThLdms3WnM0NmN4V0ZLQmx2RVpTcmFDVXRMMFVuTTBhWFl4RlJNb1JtZHZDTFdpUXkrZGRwNXpiOXNjaAppYlh3Szlrbi9VYU45Mk91UjZ4bzVTNFFHU0Y1QWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJRRHd5eGhpMGRLbXV1VkNUTURscWg2dGlYRVBqQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQ3Y2cTdGNkw5dQpDaVBZZU1yYzdZZEZ1VGNnMWVQbXoyZkREcTNKZmJLajNCTGJTd3ZoL1pRMjh4cVV5clh2anM5NDVLbDdmc0JKCkMva2gzdEV4MldXUk9xbFdieVZ2UTlDYkV4T3dDTTkzS2VjUEsvdjl6S1NaU3VsdGh4VitJSGdTaVNheGs1SVAKRnJJME0xbjFYaWhDVjJ3RUMzck42NWZlMEphTjlRc2tFUVV2TG9VSk0vM0lpOFdsVTdGZFJFemwwTFVCZ2FDegpWTUc3RDdNZFBScHJuMk5LWWhMNnRmeUlxMlNpSE9rSU1VRk5zYlFZQ2p1VEJ5QUZhYjk0b1BiM2Y3d3M1dHpFClBQME1OcXpjZmJ0SUZUQkZTa1JnTHNNRTg5KzhPVWxtU2taSW96cVhLNFNCbElqcFlnb0w2UnRTK0ZWeERVS3cKNElMS05RY0JFa1lqCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K', credentialsId: 'kubeconfig', serverUrl: 'https://10.48.228.66:6443') {
						sh 'kubectl apply -f deployment.yaml'
					}
				}
			}
		}
	}

	post {
		success {
			echo 'Build&Deploy completed succesfully!'
		}
		failure {
			echo 'Build&Deploy failed. Check logs.'
		}
	}
}
