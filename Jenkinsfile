stage('Build & Test') {
    agent { label 'build' }   // matches the “Labels” field above
    steps {
        checkout scm          // pulls the test branch
        sh 'mvn -B clean verify'
    }
}

