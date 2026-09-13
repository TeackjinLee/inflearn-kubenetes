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

에러메세지 확인 방법
```bash
kubectl describe pods [파드명]
kubectl describe pods spring-pod-1
```
로그 확인 방법
```bash
kubectl logs [파드명]
kubectl logs spring-pod-1
```

디플로이먼트(Deployment)란?
파드를 묶음으로 쉽게 관리할 수 있는 기능.
- 파드의 수를 지정하는 대로 여러 개의 파드를 쉽게 생성할 수 있음
- 파드가 비정상적으로 종료된 경우, 알아서 새로 파드를 생성해 파드 수를 유지
- 동일한 구성의 여러 파드를 일괄적으로 일시 중지, 삭제, 업데이트를 하기기 쉽다.

deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: spring-deployment

# Deployment 세부 정보
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend-app

  # 배포할 Pod의 정보
  template:
    metadata:
      labels:
        app: backend-app
    spec:
      containers:
        - name: spring-container
          image: spring-server
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080


```

deployment 적용
```bash
kubectl apply -f spring-deployment.yml
```
deployment 조회
```bash
kubectl get deployment
```

디플로이먼트가 생성한 레플리카셋(ReplicaSet) 정보를 확인
```bash
kubectl get replicaset
```


서비스(Service)란?
외부로부터 들어오는 트래픽을 받아, 파드에 균등하게 분배해주는 로드밸런서 역할을 하는 기능
- 실제 서비스에서 파드에 요청을 보낼때, 포트포워드(port-forwrard)나 파드 내로 직접 접근 해서 보내지 않는다. 서비스(Service)를 통해 요청을 보내는 게 일반적이다.

1. **매니페스트 파일 추가하기**
    
**spring-service.yaml**  
```bash
    apiVersion: v1
    kind: Service
    
    # Service 기본 정보
    metadata:
      name: spring-service # Service 이름
      
    # Service 세부 정보
    spec:
      type: NodePort # Service의 종류
      selector:
        app: backend-app # 실행되고 있는 파드 중 'app: backend-app'이라는 값을 가진 파드와 서비스를 연결
      ports:
        - protocol: TCP # 서비스에 접속하기 위한 프로토콜
          port: 8080 # 쿠버네티스 내부에서 Service에 접속하기 위한 포트 번호
          targetPort: 8080 # 매핑하기 위한 파드의 포트 번호
          nodePort: 30000 # 외부에서 사용자들이 접근하게 될 포트 번호
```
    
- `Service` 종류에 대해 한 번 짚고 넘어가자. 우선 아래 3가지 개념에 대해서만 이해하고 넘어가자.
- `NodePort` : 쿠버네티스 내부에서 해당 서비스에 접속하기 위한 포트를 열고 **외부에서 접속 가능**하도록 한다.
- `ClusterIP` : 쿠버네티스 내부에서만 통신할 수 있는 IP 주소를 부여. 외부에서는 요청할 수 없다.
- `LoadBalancer` : 외부의 로드밸런서(AWS의 로드밸런서 등)를 활용해 외부에서 접속할 수 있도록 연결한다.

service 생성
```bash
kubectl apply -f spring-service.yaml
```
모든 pod,service,deployment 삭제
```bash
kubectl delete all --all
```

36. [예제] 백엔드(Spring Boot) 서버에 환경변수 등록해 사용하기
```bash
apiVersion: apps/v1
kind: Deployment

metadata:
  name: spring-deployment

spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend-app

  template:
    metadata:
      labels:
        app: backend-app
    spec:
      containers:
        - name: spring-container
          image: spring-server
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
          # 환경변수
          env:
            - name: MY_ACCOUNT
              value: [아이디]
            - name: MY_PASSWORD
              value: [비밀번호]
```

37. 컨피그맵(ConfigMap)을 활용해 환경변수 분리하기

```
apiVersion: apps/v1
kind: Deployment

# Deployment 기본 정보
metadata:
  name: spring-deployment # Deployment 이름

# Deployment 세부 정보
spec:
  replicas: 3 # 생성할 파드의 복제본 개수
  selector:
    matchLabels:
      app: backend-app # 아래에서 정의한 Pod 중 'app: backend-app'이라는 값을 가진 파드를 선택

  # 배포할 Pod 정의
  template:
    metadata:
      labels: # 레이블 (= 카테고리)
        app: backend-app
    spec:
      containers:
        - name: spring-container # 컨테이너 이름
          image: spring-server # 컨테이너를 생성할 때 사용할 이미지
          imagePullPolicy: IfNotPresent # 로컬에서 이미지를 먼저 가져온다. 없으면 레지스트리에서 가져온다.
          ports:
            - containerPort: 8080  # 컨테이너에서 사용하는 포트를 명시적으로 표현
          env:
            - name: MY_ACCOUNT
              valueFrom:
                configMapKeyRef:
                  name: spring-config # ConfigMap의 이름
                  key: my-account # ConfigMap에 설정되어 있는 Key값
            - name: MY_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: spring-config
                  key: my-password
```

ConfigMap 매니페스트 파일 생성하기
**spring-config.yaml**

```bash
apiVersion: v1
kind: ConfigMap

# ConfigMap 기본 정보
metadata:
  name: spring-config # ConfigMap 이름

# Key, Value 형식으로 설정값 저장
data:
  my-account: 아이디
  my-password: 비밀번호
```
```bash

```

3. **매니페스트 파일 반영하기**
    
```bash
$ kubectl apply -f spring-config.yaml
$ kubectl apply -f spring-deployment.yaml

# kubectl rollout restart deployment [디플로이먼트명]
$ kubectl rollout restart deployment spring-deployment # Deployment 재시작
```

39. 볼륨(Volume)이란?
볼륨(Volume)이란 데이터를 영속적으로 저장하기 위한 방법이다.
### ✅ 퍼시스턴트 볼륨 클레임(Persistent Volume Claim, PVC)이란?

<img width="1626" height="584" alt="image" src="https://github.com/user-attachments/assets/90018411-d2e2-489f-b320-b6290ef12ecc" />

퍼시스턴트 볼륨(PV)을 설명할 때 위와 같은 그림으로 설명했다. 실제로는 파드(Pod)가 퍼시스턴트 볼륨(PV)에 직접 연결할 수 없다. 퍼시스턴트 볼륨 클레임(PVC)이라는 중개자가 있어야 한다. 그래서 쿠버네티스 구조에서는 아래와 같은 구조로 퍼시스턴트 볼륨(PV)을 연결한다.

퍼시스턴트 볼륨 클레임(PVC)은 파드(Pod)와 퍼시스턴트 볼륨(PV) 사이에서 중개자 역할을 한다.

# 볼륨(Volume)을 활용해 MySQL 실행시키기

### ✅ 볼륨(Volume)을 활용해 MySQL 실행시키기

<aside>
🧑🏻‍🏫

데이터베이스를 확인해보면 이전에 만들었던 `new-db`라는 데이터베이스가 없어졌다. 실제 데이터베이스에서는 데이터가 없어져서는 안 된다. 데이터가 보존되도록 볼륨(Volume)을 설정해보자. 

</aside>

1. **퍼시스턴트 볼륨(PV), 퍼시스턴트 볼륨 클레임(PVC) 정의하기**
    
    **mysql-pv.yaml**
    
    ```bash
    apiVersion: v1
    kind: PersistentVolume
    
    # PersistentVolume 기본 정보
    metadata:
      name: mysql-pv # PersistentVolume 이름
    
    # PersistentVolume 세부 정보
    spec:
      storageClassName: my-storage # PV와 PVC의 storageClassName이 같다면 볼륨이 연결된다.
      capacity: 
        storage: 1Gi # 볼륨이 사용할 용량을 설정
      accessModes:
        - ReadWriteOnce # 아래 hostPath 타입 활용 시 이 옵션만 사용 가능
      hostPath: # hostPath 타입을 활용 (hostPath : 쿠버네티스 내부 공간을 활용)
        path: "/mnt/data" # 쿠버네티스 내부의 공간에서 /mnt/data의 경로를 볼륨으로 사용 
    ```
    
    **mysql-pvc.yaml**
    
    ```bash
    apiVersion: v1
    kind: PersistentVolumeClaim
    
    # PersistentVolumeClaim 기본 정보
    metadata:
      name: mysql-pvc # PersistentVolumeClaim 이름
      
    # PersistentVolumeClaim 세부 정보
    spec:
      storageClassName: my-storage # PV와 PVC의 storageClassName이 같다면 볼륨이 연결된다.
      accessModes:
        - ReadWriteOnce # 볼륨에 접근할 때의 권한
      resources: # PVC가 PV에 요청하는 리소스의 양을 정의
        requests: # 필요한 최소 리소스
          storage: 1Gi # PVC가 PV에 요청하는 스토리지 양 (PV가 최소 1Gi 이상은 되어야 한다.)
    ```
    

1. **기존 디플로이먼트(Deployment) 수정하기**
    
    **mysql-deploymentyaml.yaml**
    
    ```bash
    apiVersion: apps/v1
    kind: Deployment
    
    # Deployment 기본 정보
    metadata:
      name: mysql-deployment # Deployment 이름
    
    # Deployment 세부 정보
    spec:
      replicas: 1 # 생성할 파드의 복제본 개수
      selector:
        matchLabels:
          app: mysql-db # 아래에서 정의한 Pod 중 'app: backend-app'이라는 값을 가진 파드를 선택
    
      # 배포할 Pod 정의
      template:
        metadata:
          labels: # 레이블 (= 카테고리)
            app: mysql-db
        spec:
          containers:
            - name: mysql-container # 컨테이너 이름
              image: mysql # 컨테이너를 생성할 때 사용할 이미지
              ports:
                - containerPort: 3306  # 컨테이너에서 사용하는 포트를 명시적으로 표현
              env:
                - name: MYSQL_ROOT_PASSWORD
                  valueFrom:
                    secretKeyRef:
                      name: mysql-secret
                      key: mysql-root-password
                - name: MYSQL_DATABASE
                  valueFrom:
                    configMapKeyRef:
                      name: mysql-config
                      key: mysql-database
              # 컨테이너 내에서 어떤 경로를 볼륨으로 사용할 지 지정
              **volumeMounts:
    	          - name: mysql-persistent-storage** # 밑에서 설정할 volumes.name과 값이 같아야 함
    	            **mountPath: /var/lib/mysql** # mysql 컨테이너 내부에 있는 경로
          # 파드가 사용할 볼륨을 지정
          **volumes:
    	      - name: mysql-persistent-storage** # 위에서 설정할 volumeMounts.name과 일치해야 함
    	        **persistentVolumeClaim:
    	          claimName: mysql-pvc** # 연결시킬 PVC의 name과 동일해야 함
    ```
    
2. **매니페스트 파일 반영하기**
    
    ```bash
    $ kubectl apply -f mysql-pv.yaml
    $ kubectl apply -f mysql-pvc.yaml
    $ kubectl apply -f mysql-deployment.yaml
    ```

### ✅ 백엔드(Spring Boot) 서버와 MySQL 연동하기

1. **Spring Boot 프로젝트 셋팅**
    
    start.spring.io
    
    <img width="2048" height="1360" alt="image" src="https://github.com/user-attachments/assets/00bbc693-7683-4d52-bda3-9b0d9ea7c4a8" />

    - Java 17 버전을 선택하자. 아래 과정을 Java 17 버전을 기준으로 진행할 예정이다.
    - Dependencies는 `Spring Boot DevTools`, `Spring Web`, `Spring Data JPA`, `MySQL Driver`를 선택해라.
    
2. **간단한 코드 작성**
    
    **AppController**
    
    ```java
    @RestController
    public class AppController {
      @GetMapping("/")
      public String home() {
        return "Hello, World!";
      }
    }
    ```
    
3. **application.yml에 DB 연결을 위한 정보 작성하기**
    
    **application.yml**
    
    ```bash
    spring:
      datasource:
        url: jdbc:mysql://${DB_HOST}:${DB_PORT}/${DB_NAME}
        username: ${DB_USERNAME}
        password: ${DB_PASSWORD}
        driver-class-name: com.mysql.cj.jdbc.Driver
    ```
    

1. **불필요한 테스트 코드 삭제**
    
    <img width="748" height="290" alt="image" src="https://github.com/user-attachments/assets/99a01f59-878f-40d3-99cf-30ba727ed8dd" />
    
2. **Dockerfile 작성하기**
    
    **Dockerfile**
    
    ```bash
    FROM openjdk:17-jdk
    
    COPY build/libs/*SNAPSHOT.jar /app.jar
    
    ENTRYPOINT ["java", "-jar", "/app.jar"]
    ```
    
3. **Spring Boot 프로젝트 빌드하기**
    
    ```bash
    $ ./gradlew clean build
    ```
    
4. **이미지 빌드하기**
    
    ```bash
    $ docker build -t spring-server .
    ```
    
5. **스프링 서버 띄우기 위한 매니페스트 파일 작성하기**
    
    **spring-deployment.yaml**
    
    ```bash
    apiVersion: apps/v1
    kind: Deployment
    
    # Deployment 기본 정보
    metadata:
      name: spring-deployment # Deployment 이름
    
    # Deployment 세부 정보
    spec:
      replicas: 3 # 생성할 파드의 복제본 개수
      selector:
        matchLabels:
          app: backend-app # 아래에서 정의한 Pod 중 'app: backend-app'이라는 값을 가진 파드를 선택
    
      # 배포할 Pod 정의
      template:
        metadata:
          labels: # 레이블 (= 카테고리)
            app: backend-app
        spec:
          containers:
            - name: spring-container # 컨테이너 이름
              image: spring-server # 컨테이너를 생성할 때 사용할 이미지
              imagePullPolicy: IfNotPresent # 로컬에서 이미지를 먼저 가져온다. 없으면 레지스트리에서 가져온다.
              ports:
                - containerPort: 8080  # 컨테이너에서 사용하는 포트를 명시적으로 표현
              env:
                - name: DB_HOST
                  value: mysql-service # Service의 name만 입력하면 다른 서비스와 통신할 수 있다. 
    ****            - name: DB_PORT
                  value: "3306" # 숫자값을 문자로 인식하게 만들기 위해 쌍따옴표 붙여야 한다.
                - name: DB_NAME
                  value: kub-practice
                - name: DB_USERNAME
                  value: root
                - name: DB_PASSWORD
                  value: password123
    ```
    

**spring-service.yaml**

```bash
apiVersion: v1
kind: Service

# Service 기본 정보
metadata:
  name: spring-service

# Service 세부 정보
spec:
  type: NodePort # Service의 종류
  selector:
    app: backend-app # 실행되고 있는 파드 중 'app: backend-app'이라는 값을 가진 파드와 서비스를 연결
  ports:
    - protocol: TCP # 서비스에 접속하기 위한 프로토콜
      port: 8080 # 쿠버네티스 내부에서 Service에 접속하기 위한 포트 번호 (Service
      targetPort: 8080 # 매핑하기 위한 파드의 포트 번호
      nodePort: 30000 # 외부에서 사용자들이 접근하게 될 포트 번호
```

1. **매니페스트 파일 실행하기**
    
    ```bash
    $ kubectl apply -f spring-deployment.yaml
    $ kubectl apply -f spring-service.yaml
    ```
    
2. **MySQL과 연결된 백엔드 서버 잘 작동하는 지 확인하기**
    <img width="334" height="149" alt="image" src="https://github.com/user-attachments/assets/a6b8f3bd-a759-48ad-afe5-a3a12519cb9d" />

    
    
### ✅ 그림으로 이해하기
<img width="2048" height="939" alt="image" src="https://github.com/user-attachments/assets/ce0cd49b-31a1-4804-b95c-c20465140894" />
