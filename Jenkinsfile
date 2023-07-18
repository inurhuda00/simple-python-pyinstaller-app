node {

    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash name: 'compiled-results', includes: 'sources/*.py*'
        }
    }

    docker.image('qnib/pytest').inside {
        stage('Test') {
            try {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            } finally {
                junit 'test-reports/results.xml'
            }
        }
    }
    
    stage('Manual Approve') {
        node {
            input message: 'Lanjutkan ke tahap Deploy? (Klik "Proceed" untuk lanjut)'
        }
    }
    
    stage('Deploy') {
        withEnv(['volume=${WORKSPACE}/sources:/src', 'image=cdrx/pyinstaller-linux:python2']) {
            node {
                unstash name: 'compiled-results'
                sh "docker run --rm -v ${volume} ${image} pyinstaller -F /src/add2vals.py"

                archiveArtifacts "sources/dist/add2vals"
                sh "docker run --rm -v ${volume} ${image} rm -rf /src/build /src/dist"
            }
        }
    }

}