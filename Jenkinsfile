pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes-lab3'
            namespace 'ns-juan-pablo-morales-hasard'
            defaultContainer 'node-tool'
            yamlFile 'agent.yaml'
        }
    }
    options {
        disableConcurrentBuilds()
        timeout(time: 40, unit: 'MINUTES')
    }
    environment {
        DH_REPO = 'jpmorales2985/entrega_juan_pablo_morales'
        GH_REPO = 'ghcr.io/juanhasard/entrega_juan_pablo_morales'
        IMAGE_TAG = 'juan-pablo-morales-hasard'
        APP_VERSION = '3.0.0'
        K8S_NAMESPACE = 'ns-juan-pablo-morales-hasard'
        DEPLOYMENT = 'app-juan-pablo-morales-hasard'
        SERVICE = 'svc-juan-pablo-morales-hasard'
    }
    stages {
        stage('install') {
            steps {
                sh 'npm install --global pnpm@10'
                sh 'node --version'
                sh 'pnpm --version'
                sh 'pnpm install --frozen-lockfile'
                sh 'mkdir -p evidencias/pipeline'
            }
        }
        stage('test') {
            steps {
                sh 'pnpm test --runInBand'
                sh 'pnpm run test:e2e --runInBand'
            }
        }
        stage('build') {
            steps {
                sh 'pnpm build'
            }
        }
        stage('push') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'dockerhub-lab3', usernameVariable: 'DH_USER', passwordVariable: 'DH_TOKEN'),
                    usernamePassword(credentialsId: 'ghcr-lab3', usernameVariable: 'GH_USER', passwordVariable: 'GH_TOKEN')
                ]) {
                    // BuildKit lee config.json, generado con las credenciales de Jenkins.
                    sh '''
                        set +x
                        set -eu
                        node <<'NODE'
const fs = require('node:fs');
const auth = (user, token) => Buffer.from(user + ':' + token).toString('base64');
const config = { auths: {
    'https://index.docker.io/v1/': { auth: auth(process.env.DH_USER, process.env.DH_TOKEN) },
    'ghcr.io': { auth: auth(process.env.GH_USER, process.env.GH_TOKEN) }
}};
fs.writeFileSync('/docker-config/config.json', JSON.stringify(config), { mode: 0o600 });
fs.chownSync('/docker-config/config.json', 1000, 1000);
NODE
                    '''
                }
                container('buildkit') {
                    sh '''
                        set -eu
                        test -s "$DOCKER_CONFIG/config.json"
                        buildctl-daemonless.sh build \
                            --frontend dockerfile.v0 \
                            --local context=. \
                            --local dockerfile=. \
                            --output "type=image,\\\"name=${DH_REPO}:${IMAGE_TAG},${DH_REPO}:${APP_VERSION}\\\",push=true"

                        buildctl-daemonless.sh build \
                            --frontend dockerfile.v0 \
                            --local context=. \
                            --local dockerfile=. \
                            --output "type=image,\\\"name=${GH_REPO}:${IMAGE_TAG},${GH_REPO}:${APP_VERSION}\\\",push=true"
                    '''
                }
            }
            post {
                always {
                    sh 'rm -f /docker-config/config.json'
                }
            }
        }
        stage('deploy') {
            steps {
                container('kubectl-tool') {
                    sh '''
                        set -eu
                        kubectl -n "$K8S_NAMESPACE" set image "deployment/$DEPLOYMENT" "api=$DH_REPO:$IMAGE_TAG"
                        # Renovar los pods porque se reutiliza la etiqueta del nombre.
                        kubectl -n "$K8S_NAMESPACE" rollout restart "deployment/$DEPLOYMENT"
                        kubectl -n "$K8S_NAMESPACE" rollout status "deployment/$DEPLOYMENT" --timeout=300s
                        kubectl -n "$K8S_NAMESPACE" get deployment,pods,svc -o wide > evidencias/pipeline/recursos.txt
                        kubectl -n "$K8S_NAMESPACE" logs "deployment/$DEPLOYMENT" --tail=100 > evidencias/pipeline/aplicacion.log
                        curl --fail --silent --show-error --retry 12 --retry-connrefused --retry-delay 5 --max-time 10 \
                            "http://${SERVICE}.${K8S_NAMESPACE}.svc.cluster.local/lab" \
                            -o evidencias/pipeline/respuesta-lab.json
                    '''
                }
                sh '''
                    node <<'NODE'
const fs = require('node:fs');
const data = JSON.parse(fs.readFileSync('evidencias/pipeline/respuesta-lab.json', 'utf8'));
for (const key of ['AMBIENTE', 'API_KEY']) {
    if (typeof data[key] !== 'string' || !data[key].trim() || data[key] === 'SIN COMPLETAR') {
        throw new Error('Variable ausente o sin configurar: ' + key);
    }
}
console.log('GET /lab correcto: AMBIENTE y API_KEY configurados');
NODE
                '''
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'evidencias/pipeline/**', allowEmptyArchive: true
        }
    }
}
