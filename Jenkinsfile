pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            agent: maven-agent
        spec:
          containers:
          - name: maven
            image: maven:alpine
            command:
            - cat
            tty: true
            
          - name: git
            image:  bitnami/git
            command:
            - cat
            tty: true

          - name: kaniko
            image: gcr.io/kaniko-project/executor:debug
            command:
            - sleep
            args:
            - 99999
            volumeMounts:
            - name: docker-secret
              mountPath: /kaniko/.docker/
          volumes:
          - name: docker-secret
            secret:
              secretName: docker-secret
              items:
              - key: .dockerconfigjson
                path: config.json
        '''
    }
  }
  stages {

    stage('Maven Build') {
		when {
			anyOf {
				branch 'master'
				branch 'feature/*'
        branch 'release/*'
			}
		}
		steps {
		container('maven') {
			sh 'mvn clean install'
			}
		}
		}

    stage('Maven Test') {
		when {
			anyOf {
				branch 'master'
				branch 'feature/*'
        branch 'release/*'
			}
		}
		steps {
		container('git') {
        script {
          TAG_NO = sh (
            script: "git tag -l  | head -1",
            returnStdout: true ).trim()
          if (env.BRANCH_NAME == 'master') {
            IMAGE_ID = TAG_NO
          }else {
            IMAGE_ID = TAG_NO + "-SNAPSHOT"
          }
        }
			}
		}
    }

    stage('Docker Build') {
		when {
			anyOf {
				branch 'master'
				branch 'release/*'		
			}
		}
		steps {
		container(name :'kaniko', shell: '/busybox/sh' ) {

			sh '''
				#!/busybox/sh
        
        ls -lR /home/jenkins/agent
				/kaniko/executor  --destination mohamedhani/onlinebookstore:${IMAGE_ID}  --context /home/jenkins/agent/workspace/ookstore-pipline_release_testing/
				'''
			}
		}
    }

  }
}