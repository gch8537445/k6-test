pipeline {
    agent any

    parameters {
        string(name: 'VUS', defaultValue: '10', description: '虚拟用户数')
        string(name: 'DURATION', defaultValue: '30s', description: '测试持续时间')
        string(name: 'TEST_SCRIPT', defaultValue: 'script.js', description: '测试脚本文件名')
    }

    stages {
        /* stage('准备测试环境') {
            steps {
                sh 'docker pull grafana/k6:1.0.0'
            }
        } */

        stage('执行k6测试') {
            steps {
                script{
                    // 1. 获取 Jenkins 容器内的工作区路径
                    def jenkinsContainerWorkspace = WORKSPACE

                    // 2. 定义 Jenkins 容器内 jenkins_home 的路径 和 宿主机上对应的 jenkins_home 路径
                    def jenkinsContainerHome = "/var/jenkins_home"
                    def hostJenkinsHome = "/data/docker_volume/jenkins" // 这个是你 docker run jenkins 时 -v 的宿主机路径

                    // 3. 计算出在宿主机上对应的工作区路径
                    // 将容器内 WORKSPACE 路径中的 /var/jenkins_home 替换为宿主机上的路径
                    def hostWorkspacePath = jenkinsContainerWorkspace.replaceFirst("^${jenkinsContainerHome}", hostJenkinsHome)

                    sh "echo 'Jenkins Container Workspace: ${jenkinsContainerWorkspace}'"
                    sh "echo 'Host machine Workspace Path for Docker mount: ${hostWorkspacePath}'"
                    sh "echo 'Listing files in Jenkins Container WORKSPACE:' && ls -la ${jenkinsContainerWorkspace}"
                    // 可以在宿主机上 ls 一下 hostWorkspacePath 确认文件存在 (需要权限)
                    // sh "ls -la ${hostWorkspacePath}" // 这行可能因为权限或路径问题在容器内执行失败，主要用于调试理解

                    sh """
                    docker run --rm -i -v "${hostWorkspacePath}:/scripts" \
                        grafana/k6:1.0.0 run \
                        -e K6_PROMETHEUS_RW_SERVER_URL=http://192.168.207.128:9090/api/v1/write
                        --output experimental-prometheus-rw \
                        --output json \
                        --summary-export=/scripts/results.json \
                        --vus ${params.VUS} \
                        --duration ${params.DURATION} \
                        /scripts/${params.TEST_SCRIPT}
                    """
                }

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