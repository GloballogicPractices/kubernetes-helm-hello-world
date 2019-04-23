def label = "kubectl-${UUID.randomUUID().toString()}"



podTemplate(label: label, yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:

  - name: helm
    image: dtzar/helm-kubectl:2.13.0
    command: ['cat']
    tty: true
  - name: docker
    image: docker:1.11
    command: ['cat']
    tty: true
    volumeMounts:
    - name: dockersock
      mountPath: /var/run/docker.sock
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
"""
  ){
  node(label) {

    def IMAGE_WITH_TAG = "globallogicpractices/opengine-base:helm-hello-world-${BUILD_NUMBER}"
    stage('Build') {
        git        branch: 'master',
          credentialsId: 'git-akopachevskyy-globallogic',
                    url: 'git@github.com:GloballogicPractices/kubernetes-helm-hello-world.git'

        container('docker') {
            withCredentials([usernamePassword(credentialsId: 'docker_registry_credentials',
                            usernameVariable: 'DOCKER_REGISTRY_USERNAME',passwordVariable: 'DOCKER_REGISTRY_PASSWORD')]) {
                sh '''
                    set +x
                    docker login --username "$DOCKER_REGISTRY_USERNAME" --password "$DOCKER_REGISTRY_PASSWORD"
                    docker build -t opengine-jenkins .
                '''
                sh "docker tag opengine-jenkins ${IMAGE_WITH_TAG}"
                sh "docker push ${IMAGE_WITH_TAG}"
                sh "docker rmi ${IMAGE_WITH_TAG}"

            }
        }
    }

    stage('Deploy') {

         git        branch: 'master',
         credentialsId: 'git-akopachevskyy-globallogic',
                    url: 'git@github.com:GloballogicPractices/kubernetes-helm-hello-world.git'

         container('helm') {

            withCredentials([file(credentialsId: 'kube-config', variable: 'KUBECONFIG')]) {
                  sh "helm upgrade --install --set image.repositoryAndTag=${IMAGE_WITH_TAG} hello-world ./helloworld-chart "
            }
        }
    }
  }
}



