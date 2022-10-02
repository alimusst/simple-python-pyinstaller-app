node {
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py' 
            stash(name: 'compiled-results', includes: 'sources/*.py*') 
        }
    }
    stage('Test') { 
        try {
            docker.image('qnib/pytest').inside {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py' 
            }        
        } finally {
            junit 'test-reports/results.xml' 
        }
    }
    stage('Manual Approval') {
        input message: 'Lanjutkan ke tahap Deploy? (klik "Proceed" untuk Deploy)'
    }
    stage('Deploy') {
        withEnv(['VOLUME=$(pwd)/sources:/src', 'IMAGE=cdrx/pyinstaller-linux:python2']) {
            dir(path: env.BUILD_ID) { 
                unstash(name: 'compiled-results') 
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                sh "git status"
                sh "curl https://cli-assets.heroku.com/install-ubuntu.sh | sh"
                sh "heroku --version"
                sleep(time:1, unit:"MINUTES")
            }

            try {
                archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals" 
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
            } catch (e) {
                throw e
            }
        }
    }
}