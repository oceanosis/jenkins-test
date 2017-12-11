pipeline {
  agent none
  
  stages {
    stage('Unit Tests') {
      agent {
	label 'apache'
  	}
      steps {
        sh 'ant -f test.xml -v'
        junit 'reports/result.xml'
      }
    }
    stage('build') {
      agent {
        label 'apache'
        }

      steps {
        sh 'ant -f build.xml -v'
      }
     post {
       success {
         archiveArtifacts artifacts: 'dist/*.jar', fingerprint:true
       }
     }

    }
    stage('deploy') {
      agent {
        label 'apache'
        }

     steps {
	sh "mkdir -p /var/www/html/rectangles/all/${env.BUILD_NAME}"
      sh "cp dist/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BUILD_NAME}/"
    }
   } 
    stage('Test on CentOS') {
      agent {
        label 'CentOS'
        }

     steps {
      sh "wget http://18.194.247.142/rectangles/all/${env.BUILD_NAME}/rectangle_${env.BUILD_NUMBER}.jar"
      sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 10 10"
    }
   }
   stage('Test on Debian') {
      agent {
	docker 'openjdk:8u121-jre'
     }
     steps {
	sh "wget http://18.194.247.142/rectangles/all/${env.BUILD_NAME}/rectangle_${env.BUILD_NUMBER}.jar"
	sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 58 84"
 	}
    }
  stage('Promote to Green'){
	agent {
          label 'apache'
        }
	when {
		branch 'master'
	}
	steps {
          sh "cp /var/www/html/rectangles/all/${env.BUILD_NAME}/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.BUILD_NUMBER}.jar"

	}
    }
    stage('Promote Development Branch to Master') {
        agent {
          label 'apache'
        }
	when {
                branch 'development'
        } 
	steps {
	   echo "Stashing Any Local Changes"
	   sh " git stash"
           echo "Checking Out Development Branch"
           sh 'git pull origin development'
	   sh "git checkout development"
	   echo "Checking out Master Branch"
	   sh "git checkout master"
	   echo "Merging Dev into Master"
	   sh "git merge development"
	   echo "Push into Master" 
   	   sh 'git push origin master'
	}   
     }
  }
}
