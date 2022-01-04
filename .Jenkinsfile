podTemplate(containers: [
    containerTemplate(name: 'golang', image: 'golang:1.16.0', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'kubectl', image: 'cnych/kubectl', command: 'cat', ttyEnabled: true),
  ],
  yaml: """\
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: uhub.service.ucloud.cn/longlong/executor:debug
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  restartPolicy: Never
  volumes:
    - name: kaniko-secret
      secret:
        secretName: regcred
    """.stripIndent()
  ) {
    node(POD_LABEL) {
        stage('Clone') {
            git url: 'https://github.com/DragonTwoYang/kankio-test.git/'
        }
        stage('Compile') {
            container('golang') {
                    sh """
                    make  
                    """
            }
        }
        stage('Build Image')
            container('kaniko') {
                sh """
                /kaniko/executor -c `pwd`/ -f `pwd`/docker/Dockerfile -d uhub.service.ucloud.cn/longlong/hello:v1.0
                """
            }
       stage('Deploy') {
         withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]){
           container('kubectl') {
           sh """
              mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config
              kubectl apply -f deploy/k8s.yaml 
            """
            
           }
         }
       }
    }
}
