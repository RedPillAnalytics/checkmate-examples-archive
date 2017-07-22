def options = '-S'
def properties = "-PbuildId=${env.BUILD_TAG}"
def checkmate = "./gradlew ${options} ${properties}"
def projectDir = 'obi/sample-12c'

pipeline {
  agent {
    label 'amzn-obi-12.2.1.2'
  }
  environment {
    ANALYTICS = credentials('services-analytics-user')
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
	sh "${checkmate} -p ${projectDir} metadataImport"
	sh "${checkmate} -p ${projectDir} featureBaselineWorkflow"
	sh "${checkmate} -p ${projectDir} featureRevisionWorkflow"
	junit "${projectDir}/build/test-groups/*/xml-reports/*.xml"
      }
    }

    stage('New Release') {

      when {
	branch 'master'
      }

      steps {
	sh "${checkmate} release -Prelease.disableChecks -Prelease.localOnly"
      }
    } // end of New Release stage

    stage('Publish Artifacts') {
      steps {
	sh "${checkmate} -p ${projectDir} publish"
      }
    } // end of Publish Artifacts stage

    stage('Publish Release') {

      when {
	branch 'master'
      }

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
