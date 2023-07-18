node {

    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
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
        withEnv(['volume=$(pwd)/sources:/src', 'image=cdrx/pyinstaller-linux:python2']) {
            node {
                
                sh "docker run --rm -v ${volume} ${image} 'pyinstaller -F add2vals.py'"
                
                sh 'sleep 1m'
            
                archiveArtifacts "sources/dist/add2vals" 
                sh "docker run --rm -v ${volume} ${image} 'rm -rf build dist'"
            }
        }
    }
    
}