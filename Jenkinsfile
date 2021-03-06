pipeline {
  agent none

  environment {
	MAJOR_VERSION = 1
  }
  
  stages {
    stage('Say Hello') {
	agent any

	steps {
		sayHello 'To Everyone that reads this!'
	}

    }
    stage('Git Info') {
	agent any

	steps {
		echo "My Branch Name: ${env.BRANCH_NAME}"
		script {
		  def myLib = new jenkinsTest.git.gitStuff();

		  echo "My Commit: ${myLib.gitCommit("${env.WORKSPACE}/.git")}"
		}
	}
   }
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
	sh "mkdir -p /var/www/html/rectangles/all/${env.BRANCH_NAME}"
      sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
    }
   } 
    stage('Test on CentOS') {
      agent {
        label 'CentOS'
        }

     steps {
      sh "wget http://18.194.247.142/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
      sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 10 10"
    }
   }
   stage('Test on Debian') {
      agent {
	docker 'openjdk:8u121-jre'
     }
     steps {
	sh "wget http://18.194.247.142/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
	sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 5 8"
 	}
    }
  stage('Promote to Green - PRODUCTION'){
	agent {
          label 'apache'
        }
	when {
		branch 'master'
	}
	steps {
          sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"

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
        sh 'git stash'
        echo "Checking Out Development Branch"
        sh 'git pull origin development'
        sh 'git checkout development'
        echo 'Checking Out Master Branch'
        sh 'git pull origin'
        sh 'git checkout master'
        echo 'Merging Development into Master Branch'
        sh 'git merge development'
        echo 'Pushing to Origin Master'
        sh 'git push origin master'
	echo 'Tagging the Release'
	sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
	sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
	}   
       post {
        success {
          emailext(
                subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] SUCCESS: Developement Promoted to Master",
                body: """<p>Check console output</p><p>'${JOB_NAME} : ${env.BUILD_URL}'</p>""",
                to: "dumluufuk@gmail.com"
          )
        }
       } 
    }
  }
     post {
	failure {
	  emailext(
		subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed",
		body: """<p>Check console output</p><p>'${JOB_NAME} : ${env.BUILD_URL}'</p>""",
		to: "dumluufuk@gmail.com"
	  )
	}
     }
}
