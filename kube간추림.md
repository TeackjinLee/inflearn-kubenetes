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
결과
```
NAME         READY   STATUS             RESTARTS   AGE
spring-pod   0/1     ImagePullBackOff   0          17m
```
19. 이미지가 없다고 에러가 뜨는 이유 (이미지 풀 정책)
ImagePullPolicy : 이미지 풀 정책 설정
  Always : 도커허브+ECR같은 원격저장소에서만 이미지 가져옴 
  IfNotPresent : 로컬에서 먼저 -> 원격저장소
  Never : 로컬에서만 이미지 가져옴

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
        - containerPort: 8080 # 명시적 문서
      imagePullPolicy: IfNotPresent
```

백엔드(Spring Boot) 서버 3개 띄워보기
- 트래픽 증가 서버 버벅거림을 수평적 확장(서버의 개수를 늘리는 방식)을 통해 해결한다.

spring-pod-1,2,3 이런식으로 증가 그러나 100개인경우는? 이런경우는 depolyment로 관리.
```
---
apiVersion: v1
kind: Pod

metadata:
  name: spring-pod-1

spec:
  containers:
    - name: spring-container
      image: spring-server
      ports:
        - containerPort: 8080 # 명시적 문서
      imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Pod

metadata:
  name: spring-pod-2

spec:
  containers:
    - name: spring-container
      image: spring-server
      ports:
        - containerPort: 8080 # 명시적 문서
      imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Pod

metadata:
  name: spring-pod-3

spec:
  containers:
    - name: spring-container
      image: spring-server
      ports:
        - containerPort: 8080 # 명시적 문서
      imagePullPolicy: IfNotPresent
---
```



