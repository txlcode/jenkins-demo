node('haimaxy-jnlp') {
    stage('Prepare') {
        echo "1.Prepare Stage"
        echo "${git_branch}"
        checkout scm
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            build_branch = sh(returnStdout: true, script: 'git symbolic-ref --short -q HEAD').trim()
            if ( git_branch != 'master') {
                build_tag = "${git_branch}-${build_tag}"
            }
        }
        echo "$build_branch"
    }
    stage('Test') {
      echo "2.Test Stage"
    }
    stage('Build') {
        echo "3.Build Docker Image Stage"
        sh "docker build -t registry-vpc.cn-hangzhou.aliyuncs.com/cdsb/jenkins-demo:${build_tag} ."
    }
    stage('Push') {
        echo "4.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
            sh "docker login -u ${dockerHubUser} registry-vpc.cn-hangzhou.aliyuncs.com -p ${dockerHubPassword}"
            sh "docker push registry-vpc.cn-hangzhou.aliyuncs.com/cdsb/jenkins-demo:${build_tag}"
        }
    }
    stage('Deploy') {
        echo "5. Deploy Stage"
        if (${git_branch} == 'master') {
            input "确认要部署线上环境吗？"
        }
        sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
        sh "sed -i 's/<BRANCH_NAME>/${build_branch}/' k8s.yaml"
        sh "kubectl apply -f k8s.yaml --record"
    }
}
