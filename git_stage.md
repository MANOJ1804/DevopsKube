node {
    stage('Pull GIT code'){
        git 'https://github.com/MANOJ1804/DevopsKube.git'
    }
     stage('Copy File from jenkins to Anisble'){
        sh 'scp /var/lib/jenkins/workspace/devops-project/Dockerfile ubuntu@172.31.32.138:/home/ubuntu/'
        sh 'scp /var/lib/jenkins/workspace/devops-project/ansible.yml ubuntu@172.31.32.138:/home/ubuntu/'
        //sh 'scp /var/lib/jenkins/workspace/devops-project/deployment.yml ubuntu@172.31.88.152:/home/ubuntu/'
       // sh 'scp /var/lib/jenkins/workspace/devops-project/service.yml ubuntu@172.31.88.152:/home/ubuntu/'
    }
    stage('Create a Docker Image with tag'){
        sh 'ssh ubuntu@172.31.32.138 cd /home/ubuntu/'
        sh 'ssh ubuntu@172.31.32.138 docker image build -t manojkumarab/$JOB_NAME:v1.$BUILD_ID .'
        sh 'ssh ubuntu@172.31.32.138 docker image build -t manojkumarab/$JOB_NAME:latest .'
    }
    stage('Push Image to Docker Hub'){
        withCredentials([string(credentialsId: 'dockerhub_passwd', variable: 'dockerhub_passwd')]) {
          sh 'ssh ubuntu@172.31.32.138 docker login -u manojkumarab -p ${dockerhub_passwd}'
          sh 'ssh ubuntu@172.31.32.138 docker image push manojkumarab/$JOB_NAME:v1.$BUILD_ID'
          sh 'ssh ubuntu@172.31.32.138 docker image push manojkumarab/$JOB_NAME:latest'
      }
    }
    stage('Docker logout and remove Images'){
        sh 'ssh ubuntu@172.31.32.138 cd /home/ubuntu/'
        sh 'ssh ubuntu@172.31.32.138 docker image prune -af'
        sh 'ssh ubuntu@172.31.32.138 docker logout'
    }
    stage('Execute K8s from Ansible'){
        sh 'ssh ubuntu@172.31.32.138 cd /home/ubuntu/'
        sh 'ssh ubuntu@172.31.32.138 ansible-playbook ansible.yml'
    }
}

