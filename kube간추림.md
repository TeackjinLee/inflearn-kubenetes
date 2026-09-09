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

pod 조회
```bash
kubectl get pods
```

pod 접속
```bash
kubectl exec -it nginx-pod -- bash
```

pod port 포워드
```bash
# kubectl port-forword pod/[파드명] [로컬에서 포트]/파드에서 포트]
sudo kubectl port-forward pod/nginx-pod 80:80
```
pod port 삭제
```bash
kubectl delete pod nginx-pod
```

Dockerfile
```
FROM eclipse-temurin:17-jdk

COPY build/libs/*SNAPSHOT.jar app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

매니페스트 파일 (spring-pod.yaml)
쿠버네티스에게 "이런 Pod를 만들어줘"라고 지시하는 YAML 형식의 설정 파일입니다.

```
apiVersion: v1
kind: Pod
metadata:
  name: spring-pod
spec:
  containers:
    - name: spring-container
      image: spring-server
      ports:
        - containerPort: 8080
```

작성한 매니페스트 파일(spring-pod.yaml)을 적용하여 Pod를 생성
```bash
kubectl apply -f spring-pod.yaml
```

