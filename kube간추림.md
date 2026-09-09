쿠버네티스(Kubernetes)란? / 장점
  - 쿠버네티스는 다수의 컨테이너를 효율적으로 배포, 확장 및 관리 하기 위한 오픈 소스 시스템이다.
  - Docker Compose와 비슷한 느낌.

파드(Pod)란?
  - 하나의 프로그램을 실행시키는 단위를 파드(Pod)
  - 쿠버네티스에서 가장 작은 단위
  - 일반적으로 하나의 파드가 하나의 컨테이너를 가진다.(예외적으로 여러게인 경우도 있음)

파드 생성 .yaml파일
```yaml
  apiVersion: v1    # pod라는거를 기제할때는 v1으로 작성
kind: Pod

metadata:
  name: nginx-pod

spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80 # 문서화 EXPOSE 가독성을 위한 포트
```
pod 생성
```bash
kubectl apply -f new-nginx-pod.yaml
```
