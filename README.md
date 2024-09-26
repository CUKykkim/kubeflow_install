# kubeflow 로 분석 플랫폼 구축하기

## kubernetes 버전 확인

- kubernetes 1.29 버전 이상에서 동작

 - [Docker 공식홈페이지](https://desktop.docker.com/win/main/amd64/135262/Docker%20Desktop%20Installer.exe?_gl=1*1nmfr0b*_gcl_au*NDcwODQ0NTY0LjE3MjQ5MDc0Nzc.*_ga*MjQxMTA3MDcuMTY5MDI1NDEzMQ..*_ga_XJWPQMJYHQ*MTcyNTM0MTE2NS4xNC4xLjE3MjUzNDEyNjguNjAuMC4w)에서 Docker Desktop 다운로드
   
   * Docker Desktop Ver.4.270 

## 시스템 최소사양

- 4 코어, 16GB RAM 이상


## kubeflow를 설치하기 전 볼륨 설정 

- Local Path Provisioner 설치

  ```
  kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml

  ```

- Local Path Provisioner 설치 확인

  ```
  kubectl get pods -n local-path-storage

  ```

- Local Path Provisioner로 Storage Class가 생성되었는지 확인

  ```
  kubectl get storageclass
  ```
- local-path를 default Storage Class로 설정

  ```
  kubectl patch storageclass local-path -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "true"}}}'

  ```
- 변경 사항 확인

  ```
  kubectl get storageclass
  ```



-기존 hostpath의 default 설정 제거

```
kubectl patch storageclass hostpath -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": null}}}'
```


## kustomize 다운 받기

- windows terminal을 열어 ubuntu tab을 생성한다.

- 명령어를 입력해 kustomize 를 다운 받는다.


  ```
  VERSION=5.2.1
  curl -LO https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize/v${VERSION}/kustomize_v${VERSION}_linux_amd64.tar.gz

  ```

- 압축 해제 및 설치

  ```
  tar -zxvf kustomize_v${VERSION}_linux_amd64.tar.gz
  ```

- 실행 파일을 시스템 경로로 이동시키고 권한을 부여한다. 

  ```
  sudo mv kustomize /usr/local/bin/kustomize
  sudo chmod +x /usr/local/bin/kustomize
  ```


## kubeflow 설치하기

- kubeflow Kubeflow 설치에 필요한 **매니페스트 파일(manifest files) 다운

  ```
  git clone https://github.com/kubeflow/manifests.git
  cd manifests/
  ```

- `kustomize` 명령어를 입력하여 모든 yaml파일을 k8s상에서 수행
  ```
  while ! kustomize build example | kubectl apply -f -; do echo "Retrying to apply resources"; sleep 10; done
  ```

- 설치가 다 끝났다면, 다음 명령어를 사용하여 k8s 시스템상에 pod의 상태를 확인한다.

```
kubectl get po -A
```

- 설치가 완료되면 아래와 같은 결과가 나온다.

```
NAMESPACE                   NAME                                                        READY   STATUS             RESTARTS   AGE
auth                        dex-5ddf47d88d-82l5c                                        1/1     Running            1          75m
cert-manager                cert-manager-7dd5854bb4-bm2tq                               1/1     Running            0          75m
cert-manager                cert-manager-cainjector-64c949654c-wx66q                    1/1     Running            3          75m
cert-manager                cert-manager-webhook-6b57b9b886-cgvt9                       1/1     Running            0          75m
istio-system                authservice-0                                               1/1     Running            0          75m
istio-system                cluster-local-gateway-75cb7c6c88-9pwrw                      1/1     Running            0          75m
istio-system                istio-ingressgateway-79b665c95-pknck                        1/1     Running            0          75m
istio-system                istiod-86457659bb-29sb2                                     1/1     Running            0          75m
knative-eventing            eventing-controller-79895f9c56-h9gsl                        1/1     Running            0          75m
knative-eventing            eventing-webhook-78f897666-d5c5j                            1/1     Running            0          75m
knative-eventing            imc-controller-688df5bdb4-4bpdf                             1/1     Running            0          75m
knative-eventing            imc-dispatcher-646978d797-6cxlv                             1/1     Running            0          75m
knative-eventing            mt-broker-controller-67c977497-jhzsc                        1/1     Running            0          75m
knative-eventing            mt-broker-filter-66d4d77c8b-qs6ct                           1/1     Running            0          75m
knative-eventing            mt-broker-ingress-5c8dc4b5d7-qt26s                          1/1     Running            0          75m
knative-serving             activator-7476cc56d4-ngn67                                  2/2     Running            1          74m
knative-serving             autoscaler-5c648f7465-5mzf7                                 2/2     Running            1          74m
knative-serving             controller-57c545cbfb-tqwlv                                 2/2     Running            1          74m
knative-serving             istio-webhook-578b6b7654-kpmml                              2/2     Running            1          74m
knative-serving             networking-istio-6b88f745c-ptq22                            2/2     Running            1          74m
knative-serving             webhook-6fffdc4d78-2x5fh                                    2/2     Running            1          74m
kube-system                 coredns-558bd4d5db-57xg2                                    1/1     Running            0          77m
kube-system                 coredns-558bd4d5db-mctm6                                    1/1     Running            0          77m
kube-system                 etcd-docker-desktop                                         1/1     Running            0          77m
kube-system                 kube-apiserver-docker-desktop                               1/1     Running            0          77m
kube-system                 kube-controller-manager-docker-desktop                      1/1     Running            0          77m
kube-system                 kube-proxy-b9hkn                                            1/1     Running            0          77m
kube-system                 kube-scheduler-docker-desktop                               1/1     Running            2          77m
kube-system                 storage-provisioner                                         1/1     Running            3          76m
kube-system                 vpnkit-controller                                           1/1     Running            3          76m
kubeflow-user-example-com   hellokubeflow-2-r5dd4-433021892                             0/2     Completed          0          30m
kubeflow-user-example-com   ml-pipeline-ui-artifact-5dd95d555b-ht5ww                    2/2     Running            0          68m
kubeflow-user-example-com   ml-pipeline-visualizationserver-6b44c6759f-vccd5            2/2     Running            0          68m
kubeflow                    admission-webhook-deployment-7c6d8876db-w7f48               1/1     Running            0          74m
kubeflow                    cache-deployer-deployment-79fdf9c5c9-6hjh7                  2/2     Running            1          74m
kubeflow                    cache-server-5bdf4f4457-6skz9                               2/2     Running            0          74m
kubeflow                    centraldashboard-6f56b874f9-p9mlm                           1/1     Running            0          74m
kubeflow                    jupyter-web-app-deployment-7d4bfc79b-7cjtg                  1/1     Running            0          74m
kubeflow                    katib-controller-557f4f848-ph5f9                            1/1     Running            0          74m
kubeflow                    katib-db-manager-5cb7c77df8-p4h6d                           1/1     Running            0          74m
kubeflow                    katib-mysql-7894994f88-dpt2g                                1/1     Running            0          74m
kubeflow                    katib-ui-8cc499fcf-tfsvm                                    1/1     Running            0          74m
kubeflow                    kfserving-controller-manager-0                              2/2     Running            0          73m
kubeflow                    kfserving-models-web-app-db6695f6-pjg4t                     2/2     Running            0          74m
kubeflow                    kubeflow-pipelines-profile-controller-7b947f4748-dgkkh      1/1     Running            0          74m
kubeflow                    metacontroller-0                                            1/1     Running            0          73m
kubeflow                    metadata-envoy-deployment-5b4856dd5-gdchz                   1/1     Running            0          74m
kubeflow                    metadata-grpc-deployment-6b5685488-c72pq                    2/2     Running            3          74m
kubeflow                    metadata-writer-548bd879bb-flwcn                            2/2     Running            1          74m
kubeflow                    minio-5b65df66c9-4h77p                                      2/2     Running            0          74m
kubeflow                    ml-pipeline-8c4b99589-h6vrn                                 2/2     Running            1          74m
kubeflow                    ml-pipeline-persistenceagent-d6bdc77bd-zq2gn                2/2     Running            0          74m
kubeflow                    ml-pipeline-scheduledworkflow-5db54d75c5-272tw              2/2     Running            0          74m
kubeflow                    ml-pipeline-ui-5bd8d6dc84-5mnxg                             2/2     Running            0          74m
kubeflow                    ml-pipeline-viewer-crd-68fb5f4d58-fgljw                     2/2     Running            1          74m
kubeflow                    ml-pipeline-visualizationserver-8476b5c645-7w64q            2/2     Running            0          74m
kubeflow                    mpi-operator-d5bfb8489-bvz6s                                1/1     Running            17         74m
kubeflow                    mysql-f7b9b7dd4-h9trv                                       2/2     Running            0          74m
kubeflow                    notebook-controller-deployment-8b64878d4-dqw7w              1/1     Running            0          74m
kubeflow                    profiles-deployment-64675b488b-g5k9k                        2/2     Running            0          74m
kubeflow                    tensorboard-controller-controller-manager-9ff658cf8-q2szk   3/3     Running            4          74m
kubeflow                    tensorboards-web-app-deployment-754f55996d-z76cq            1/1     Running            0          74m
kubeflow                    training-operator-868876bcc8-vtglh                          1/1     Running            0          74m
kubeflow                    volumes-web-app-deployment-6dcbf595b5-74b8n                 1/1     Running            0          74m
kubeflow                    workflow-controller-5cbbb49bd8-cgkth                        2/2     Running            3          74m
```

## kubeflow 접속 하기

**다음 둘 중 하나의 방식을 사용한다.**


+ port forward 방식

```
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
```

+ End Point를 외부로 노출하는 방식

```
kubectl edit svc istio-ingressgateway -n istio-system
```

# 대시 보드에 접속하기

초기에 비밀번호는 다음과 같다.

```
user@example.com
12341234
```


## kubeflow pipeline 돌려보기

# kfp 패키지 설치
- !pip install kfp


- hello pipeline

```
import kfp
from kfp import dsl

# 첫 번째 컴포넌트: 숫자 더하기
@dsl.component
def add_numbers(a: int, b: int) -> int:
    return a + b

# 두 번째 컴포넌트: 숫자 곱하기
@dsl.component
def multiply_numbers(x: int, y: int) -> int:
    return x * y

# 파이프라인 정의
@dsl.pipeline(
    name='Hello Pipeline',
    description='먼저 두 숫자를 더하고, 그 다음 두 숫자를 곱하는 파이프라인입니다.'
)
def add_and_multiply_pipeline(a: int = 5, b: int = 3, x: int = 4, y: int = 2):
    # 첫 번째 컴포넌트
    addition_task = add_numbers(a=a, b=b)
    
    # 두 번째 컴포넌트가 첫 번째 컴포넌트 이후에 실행되도록 설정 (데이터 의존성 없음)
    multiplication_task = multiply_numbers(x=x, y=y)
    multiplication_task.after(addition_task)

# 파이프라인 컴파일
if __name__ == '__main__':
    kfp.compiler.Compiler().compile(pipeline_func=add_and_multiply_pipeline, package_path='add_and_multiply_pipeline.yaml')
```