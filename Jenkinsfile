pipeline {
    agent { label 'node' }

    parameters {
        string(name: 'SHARDS', defaultValue: '7', description: 'How many shards should we use? (enter number, job will fail with string')
    }
    environment {
        REPORT_FILES = "index1.html"
        REPORT_TITLES = "Shard 1"
    }
    stages {
        stage('Many tests') {
            steps {
                script {
                    generateReportFiles()
                    generateReportTitles()
                    doDynamicParallelTestSteps()
                }
            }
        }
        stage('Make report') {
            steps {
                script {
                    doUnstashShards()
                }
                publishHTML([
                            allowMissing: false,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: 'playwright-report',
                            reportFiles: REPORT_FILES,
                            reportName: "aggregated",
                            reportTitles: REPORT_TITLES
                        ])
            }
        }
    }
}
def generateReportFiles() {
    int totalShards = Integer.parseInt(params.SHARDS)
    for (i = 1; i < totalShards; i++) {
        int shardNum = i + 1
        REPORT_FILES = REPORT_FILES + ', index' + shardNum + '.html'
    }
}
def generateReportTitles() {
    int totalShards = Integer.parseInt(params.SHARDS)
    for (i = 1; i < totalShards; i++) {
        int shardNum = i + 1
        REPORT_TITLES = REPORT_TITLES + ', Shard ' + shardNum
    }
}
def doDynamicParallelTestSteps() {
    tests = [:]
    int totalShards = Integer.parseInt(params.SHARDS)
    for (i = 0; i < totalShards; i++) {
        def shardNum = "${i+1}"
        tests["${shardNum}"] = {
            node('node') {
                stage("Shard #${shardNum}") {
                    docker.image('ci-cd/playwright:v1.17.1').inside {
                        git branch: 'master',
                            credentialsId: 'cicd',
                            url: 'https://github.com/viaacode/playwright-demo.git'
                        catchError() {
                            sh "npx playwright test --shard=${shardNum}/${totalShards}"
                        }
                        sh "mv playwright-report/index.html playwright-report/index${shardNum}.html"
                        stash includes: "playwright-report/index${shardNum}.html", name: "shard${shardNum}"
                    }
                }
            }
        }
    }
    parallel tests
}
def doUnstashShards() {
    int totalShards = Integer.parseInt(params.SHARDS)
    for (i = 0; i < totalShards; i++) {
        unstash "shard${i+1}"
    }
}
