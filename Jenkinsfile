pipeline {
  agent any
  triggers { pollSCM('H/2 * * * *') }
  options {
    skipDefaultCheckout()
    timeout(time: 15, unit: 'MINUTES')
  }
  environment {
    IMAGE = "books-api:${env.BUILD_NUMBER}"
    // La regresion (09 y 10) espera 401: el API tiene que arrancar con la llave
    // activa y con el mismo valor que apiKey en bruno/regresion/environments/local.bru.
    DEMO_API_KEY = 's3cret'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Start API') {
      steps {
        sh '''
          nohup uv run api.py > api.log 2>&1 &
          echo $! > api.pid
          for i in $(seq 1 60); do
            curl -sf http://127.0.0.1:8000/health && exit 0
            sleep 1
          done
          cat api.log; exit 1
        '''
      }
    }
    stage('Regresion') {
      steps {
        dir('bruno/regresion') {
          sh 'bru run --env local --sandbox=developer --reporter-junit results.xml'
        }
      }
    }
    stage('Build image') {
      steps { sh 'docker build -f Containerfile -t "$IMAGE" .' }
    }
    stage('Deploy') {
      steps {
        sh '''
          docker rm -f books-api || true
          docker run -d --name books-api -p 8001:8000 "$IMAGE"
          for i in $(seq 1 30); do
            docker exec books-api python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/health')" && exit 0
            sleep 1
          done
          docker logs books-api; exit 1
        '''
      }
    }
  }
  post {
    always {
      junit allowEmptyResults: true, testResults: 'bruno/regresion/results.xml'
      sh 'kill "$(cat api.pid)" 2>/dev/null || true'
    }
  }
}
