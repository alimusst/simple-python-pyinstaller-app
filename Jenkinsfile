node {
    stage('build') {
        image 'python:2-alpine'
        sh 'python -m py_compile sources/add2vals.py sources/calc.py'
    }
}