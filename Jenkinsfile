pipeline {
    agent none

    environment {
        MAJOR_VERSION = 1
    }

    stages {
        stage('say hello') {
            agent any
            steps {
                sayHello 'Awesome Student'
            }
        }
        stage('git information') {
            agent any
            steps {
                echo "Branch name: ${env.BRANCH_NAME}"
                script {
                    def myLib = new linuxacademy.git.gitStuff();
                    echo "Commit: ${myLib.gitCommit("${env.WORKSPACE}/.git")}"
                }
            }
        }
        stage('test') {
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
                    archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
                }
            }
        }
        stage('deploy') {
            agent {
                label 'apache'
            }
            steps {
                sh "if ![ -d \"/var/www/html/rectangles/all/${env.BRANCH_NAME}\" ]; then mkdir /var/www/html/rectangles/all/${env.BRANCH_NAME}; fi"
                sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}"
            }
        }
        stage('Running on CentOS') {
            agent {
                label 'CentOs'
            }
            steps {
                sh "wget http://agoldin1.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
                sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
            }
        }
        stage('Test on Debian') {
            agent {
                docker 'openjdk:8u131-jre'
            }
            steps {
                sh "wget http://agoldin1.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
                sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
            }
        }
        stage('Promote to Green') {
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
        stage ('Promote Development Branch to Master') {
            agent {
                label 'apache'
            }
            when {
                branch 'development'
            }
            steps {
                echo "Stashing any local changes"
                sh 'git stash'
                echo "Checking out development branch"
                sh 'git checkout development'
                echo "Checking out the master branch"
                sh 'git pull origin'
                sh 'git checkout master'
                echo "Merging development into master"
                sh 'git merge development'
                echo "Pushing to origin master"
                sh 'git push origin master'
                echo "Tagging the release"
                sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
                sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
            }
            // commented out because it isn't working
//            post {
//                success {
//                    emailext(
//                        subject: "${env.JOB_NAME} [Build #${env.BUILD_NUMBER}] Development Promoted to Master",
//                        body: """<p>${env.JOB_NAME} [Build #${env.BUILD_NUMBER}] Development Promoted to Master</p>
//                        <p>Check console output at <a href="${env.BUILD_URL}>${env.JOB_NAME} [Build #${env.BUILD_NUMBER}]</a></p>""",
//                        to: "agoldin@learnvest.com"
//                    )
//                }
//            }
        }
    }
    // commented out because it isn't working
//    post {
//        failure {
//            emailext(
//                subject: "${env.JOB_NAME} [Build #${env.BUILD_NUMBER}] Failed!",
//                body: """<p>${env.JOB_NAME} [Build #${env.BUILD_NUMBER}] Failed!</p>
//                <p>Check console output at <a href="${env.BUILD_URL}>${env.JOB_NAME} [Build #${env.BUILD_NUMBER}]</a></p>""",
//                to: "agoldin@learnvest.com"
//            )
//        }
//    }
}
