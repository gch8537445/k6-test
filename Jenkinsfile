pipeline {
    agent any

    parameters {
        string(name: 'VUS', defaultValue: '10', description: '虚拟用户数')
        string(name: 'DURATION', defaultValue: '30s', description: '测试持续时间')
        string(name: 'TEST_SCRIPT', defaultValue: 'script.js', description: '测试脚本文件名')
    }

    stages {
/*         stage('准备测试环境') {
            steps {
                sh 'docker pull grafana/k6:1.0.0'
            }
        } */

        stage('执行k6测试') {
            steps {
                sh "echo 'Listing files in WORKSPACE:' && ls -la ${WORKSPACE}" // 打印工作区文件列表
                sh """
                docker run -i -v "${WORKSPACE}:/scripts" \
                    grafana/k6:1.0.0 run \
                    --out influxdb=http://192.168.207.128:8086/k6 \
                    --vus ${params.VUS} \
                    --duration ${params.DURATION} \
                    ${params.TEST_SCRIPT}
                """
            }
        }
    }

    post {
always {
            // 保存JSON结果作为构建产物
            archiveArtifacts artifacts: 'results.json', fingerprint: true

            // 添加指向Grafana仪表板的链接
            echo '查看详细的性能测试图表: http://192.168.207.128:3000/d/k6-performance'
        }
    }
}