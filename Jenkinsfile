#!groovy
import groovy.transform.Field

@Library("liferay-sdlc-jenkins-lib") import static org.liferay.sdlc.SDLCPrUtilities.*

@Field final gitRepository = 'taksan/myown'
@Field final projectName = "STARK"
@Field final projectKey  = "myown"

def onError() {
	handleError(gitRepository, "gabriel.takeuchi@objective.com.br", "github_taksan_myown")
}

node ("myown") {
	try {
		stage('Checkout') {
			checkout scm
		}

		stage('Setup') {
			if (fileExists("bundles"))
				deleteRecursive: "bundles"

			appendAdditionalCommand("build.gradle", [
				"_SONAR_PROJECT_NAME_" : projectName,
				"_SONAR_PROJECT_KEY_"  : projectKey
			]) ;
			
			gradlew 'clean'
		}

		stage('Init Bundle') {
			gradlew 'initBundle'
		}

		stage('Build') {
			try {
				gradlew 'build -x test'	
			} catch (exc) {
				onError()
				throw exc
			}
		}

		stage('Test') {
			try {
				gradlew 'test'
			} catch (exc) {
				onError()
				throw exc
			} finally {
				junit '**/build/test-results/test/*.xml'
			}
		}

		stage('Sonar') {
			if (isPullRequest()) {
				println "Will evaluate the Pull Request"
				sonarqube "-Dsonar.analysis.mode=preview -Dsonar.github.pullRequest=${CHANGE_ID} -Dsonar.github.oauth=${GithubOauth} -Dsonar.github.repository=${gitRepository}"
			}
			else
				sonarqube ""
		}
	}finally {
		stage('Cleanup') {
			dir(workspace) {
				deleteDir();
			}
		}
	}
}
