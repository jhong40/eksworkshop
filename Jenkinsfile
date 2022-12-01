pipeline {
    agent {
        kubernetes {
            label 'kaniko'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  serviceAccountName: jenkins
  containers:
  - name: kaniko
    image: xxxx.dkr.ecr.yyy.amazonaws.com/kaniko:latest
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
  restartPolicy: Never
"""
        }
    }
    stages {
        stage('Make Image') {
            environment {
                DOCKERFILE  = "Dockerfile.v3"
                GITREPO     = "git://github.com/ollypom/mysfits.git"
                CONTEXT     = "./api"
                REGISTRY    = 'xxx.dkr.ecr.yyy.amazonaws.com'
                IMAGE       = 'mysfits'
                TAG         = 'latest'
            }            
            steps {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    sh '''#!/busybox/sh
                    pwd
                    ls
                    echo ${GITREPO}
                    echo ${CONTEXT} 
                    echo ${DOCKERFILE}
                    echo \${REGISTRY}/\${IMAGE}:\${TAG} 
                    /kaniko/executor \\
                    --context=\${GITREPO} \\
                    --context-sub-path=\${CONTEXT} \\
                    --dockerfile=\${DOCKERFILE} \\
                    --destination=\${REGISTRY}/\${IMAGE}:\${TAG}    
                    '''
                }
            }
        }
    }
}
