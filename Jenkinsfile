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
      sh "cp dist/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/"
    }
   } 
    stage('Running on CentOS') {
      agent {
        label 'CentOS'
        }

     steps {
      sh "wget http://18.194.247.142/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar"
      sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 10 10"
    }
   }
   stage('Test on Debian') {
      agent {
	docker 'openjdk:8u121-jre'
     }
     steps {
	sh "wget http://18.194.247.142/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar"
	sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 58 84"
 	}
    }
  stage('Promote to Green){
	steps {
         sh "cp /var/www/html/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.BUILD_NUMBER}.jar"

	}
    }

  }
}
