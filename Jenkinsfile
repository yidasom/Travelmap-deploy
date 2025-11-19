pipeline {
    agent any

    tools {
        gradle 'gradle-8.14.3'
        jdk 'jdk-21'
    }

    parameters {
        // 배포 환경 선택
        choice(choices: ['dev', 'prod'], name: 'PROFILE', description: '배포 환경 선택')
        // DockerHub 사용자명 입력
        string(name: 'DOCKERHUB_USERNAME',  defaultValue: '', description: 'DockerHub 사용자명을 입력하세요.')
        // GitHub  사용자명 입력
        string(name: 'GITHUB_USERNAME',  defaultValue: '', description: 'GitHub  사용자명을 입력하세요.')
    }

    environment {
        GITHUB_URL = "https://github.com/yidasom/Travelmap-deploy.git"
        APP_IMAGE   = "travelmap-tester"
        APP_VERSION = "v1.0.0"
    }

    stages {

        stage('Username 확인') {
            steps {
                script {
                    if (!env.DOCKERHUB_USERNAME?.trim() || !env.GITHUB_USERNAME?.trim()) {
                        error "[파라미터와 함께 빌드]에 GITHUB_USERNAME를 본인의 username으로 입력해 주세요! 매번 입력이 번거롭다면 Jenkinsfile에서 parameters에 입력 항목을 삭제 하시고 environment에 전역값을 넣은 후 해당 조건문은 삭제해 주세요. "
                    }
                }
            }
        }

        stage('소스파일 체크아웃') {
            steps {
                // 소스코드를 가져올 Github 주소
                git branch: 'main', url: 'https://github.com/yidasom/TravelMap.git'
            }
        }

        stage('소스 빌드') {
            steps {
                // 빌드 배포가 중요한 건 아니기 때문에 실제 실행되지 않도록 echo 명령을 사용
                // 755권한 필요 (윈도우에서 Git으로 소스 업로드시 권한은 644)
                echo "chmod +x ./gradlew"
                echo "gradle clean build"
            }
        }

//         stage('릴리즈파일 체크아웃') {
//             steps {
//                 checkout scmGit(branches: [[name: '*/main']],
//                         extensions: [[$class: 'SparseCheckoutPaths',
//                                       sparseCheckoutPaths: [[path: "/deploy"]]]],
//                         userRemoteConfigs: [[url: "${GITHUB_URL}"]])
//             }
//         }

        stage('컨테이너 빌드') {
            steps {
                // jar 파일 복사
                echo "cp ./build/libs/*.jar ./deploy/build/docker/app.jar"
                // 도커 빌드
                echo "docker build -t ${DOCKERHUB_USERNAME}/${APP_IMAGE}:${APP_VERSION} ./deploy/build/docker"
            }
        }

        stage('컨테이너 업로드') {
            steps {
                // DockerHub로 이미지 업로드
                echo "docker push ${DOCKERHUB_USERNAME}/${APP_IMAGE}:${APP_VERSION}"
            }
        }

        stage('Travelmap-deploy 레포 체크아웃') {
            steps {
                git branch: 'main', url: "${DEPLOY_REPO}"
            }
        }

        stage('커스터마이즈 템플릿 확인') {
            steps {
                // K8S 배포
                sh "kubectl kustomize ./deploy/kustomize/travelmap-tester/overlays/${params.PROFILE}"
            }
        }

//         stage('커스터마이즈 배포') {
//             steps {
//                 // K8S 배포
//                 input message: '배포 시작', ok: "Yes"
//                 sh "kubectl apply -f ./deploy/kubectl/namespace-${params.PROFILE}.yaml"
//                 sh "kubectl apply -k ./deploy/kustomize/travelmap-tester/overlays/${params.PROFILE}"
//             }
//         }
    }
}