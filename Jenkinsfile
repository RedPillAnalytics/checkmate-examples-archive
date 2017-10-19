def options = '-S'
def properties = "-PbuildId=${env.BUILD_TAG}"
def checkmate = "./gradlew ${options} ${properties}"
def projectDir = 'obi/sample-12c'

pipeline {
  agent {
    label 'amzn-obi-12.2.1.2'
  }
  environment {
    // Jenkins credentials used to store sensitive information
    ADMIN = credentials('adminUser')
    repositoryPassword = credentials('repositoryPassword')
  }
  stages {

    stage('Build and Startup') {

      steps {
        // start up OBIEE while also doing the build
        parallel (
          "OBIEE Startup": {
            sh "/home/oracle/fmw/config/domains/bi/bitools/bin/start.sh"
          },
	
          // Build brokerage project
          "Brokerage Build": {
            sh "${checkmate} -p ${projectDir} featureCompare"
          }
	
	      ) // end of parallel function
      }
    } // end of Build and Startup stage

    stage('Brokerage Regression Test') {
      steps{
        // run baseline and revision, and then register the results
        sh "${checkmate} -p ${projectDir} featureBaselineWorkflow"
        sh "${checkmate} -p ${projectDir} featureRevisionWorkflow"
        junit allowEmptyResults: true, testResults: "${projectDir}/build/test-groups/*/xml-reports/*.xml"
      }
    }

    stage('New Release') {
      when {
	      branch 'master'
      }
      // create a new release when the build is against master branch
      steps {
	      sh "${checkmate} release -Prelease.disableChecks -Prelease.localOnly"
      }
    } // end of New Release stage

    // Publish distribution files to Maven
    stage('Publish Artifacts') {
      steps {
	      sh "${checkmate} -p ${projectDir} publish"
      }
    } // end of Publish Artifacts stage

    stage('Publish Release') {

      when {
	      branch 'master'
      }

      // publish new releases to GitHub as well
      // This creates a new Github release and attaches distribution files
      steps {
	      sh "${checkmate} githubRelease"
      }
    } // end of New Release stage
    
  } // end of Stages block

  post {
    always {
      archive "obiee/**/build/distributions/*"
      sh "${checkmate} -p ${projectDir} analytics"
    }
  }
}
