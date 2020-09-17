# final-assesment

구현 Repository

<img width="400" alt="스크린샷 2020-09-17 오후 1 24 48" src="https://user-images.githubusercontent.com/68723566/93420243-86271c80-f8e9-11ea-9a2e-88279282273d.png">
<img width="400" alt="스크린샷 2020-09-17 오후 1 25 30" src="https://user-images.githubusercontent.com/68723566/93420251-87f0e000-f8e9-11ea-8db5-ed01ce595944.png">

## 개인 AWS계정에 팀과제 및 개인과제 추가된 내용이 EKS 배포 됨을 확인
<img width="800" alt="스크린샷 2020-09-17 오전 11 17 32" src="https://user-images.githubusercontent.com/68723566/93420710-7cea7f80-f8ea-11ea-891e-b8b1a4b2d2fe.png">


### 1. 이벤트 스토밍

## 팀과제 완성된 모형
![image](https://user-images.githubusercontent.com/68723566/93046088-ecb2fd00-f693-11ea-836f-bd166b106df1.png)

## 개인과제 추가된 모형
고객이 받은 리워드로 교환한 gift에 대한 내역을 특정 메신저(텔레그램)를 통해 전달하는 기능 추가
![telegram](https://user-images.githubusercontent.com/68723566/93420391-c1295000-f8e9-11ea-9179-a8549d15b53f.JPG)




### 4. 게이트웨이


#istio Virtual Service에 telegram path 추가함
![path](https://user-images.githubusercontent.com/68723566/93420493-fdf54700-f8e9-11ea-9968-9195e4b00d6f.png)

#설치된 istio확인
![istio](https://user-images.githubusercontent.com/68723566/93420501-0188ce00-f8ea-11ea-8dd7-4159ae11892b.png)


#적용된 yml 확인
![yml](https://user-images.githubusercontent.com/68723566/93420504-03eb2800-f8ea-11ea-99c1-c25004d7a83e.png)


## 6. CI/CD 적용

AWS Codebuild를 통한 webhook 빌드 기능 적용
build script는 각 프로젝트 루트 경로에 buildspec.yml에 포함함
![cicd](https://user-images.githubusercontent.com/68723566/93420805-b28f6880-f8ea-11ea-9888-6739b9c864f2.png)


## 8. Autoscale out 적용


## 10. ConfigMap/Persistence Volume 적용

pvc 적용
![pvc](https://user-images.githubusercontent.com/68723566/93421409-3c8c0100-f8ec-11ea-8d6b-880fda99e3f6.png)

![pvc](https://user-images.githubusercontent.com/68723566/93423025-fa64be80-f8ef-11ea-8b4a-9e171e4dbd46.png)
![pvc](https://user-images.githubusercontent.com/68723566/93423039-fe90dc00-f8ef-11ea-829e-dbe7e248c889.png)

## 11.Polyglot 적용

* Polyglot 설정을 위해 구성한 amazon rds 의 db서버 정보를 configmap으로 구성했다.

```
...
apiVersion: v1
kind: ConfigMap
metadata:
    name: db-config
    namespace: game
data:
    DB_URL: admin08-mariadb.cuy0kl2qzoel.ap-northeast-2.rds.amazonaws.com:3306 
    DB_USER: admin
    DB_PASSWORD: skccadmin
...
```
application.yml 설정
```
...
  datasource:
    url: jdbc:mariadb://${DB_URL}/${DB_NAME}
    driver-class-name: org.mariadb.jdbc.Driver
    username: ${DB_USER}
    password: ${DB_PASSWORD}
...
```

![pvc](https://user-images.githubusercontent.com/68723566/93423708-89260b00-f8f1-11ea-89e5-0eb1bfab93a3.png)


## 12.Liveness Probe 적용

- Liveness Probe를 통해 특정 경로밑에 파일이 존재하는지 확인 하면서 컨테이너가 동작 중인지 여부를 판단한다. 

Liveness 설정 (테스트를 위해 Persistent Volume 에 미리 healthy 파일을 생성해 놨습니다.)
![image](https://user-images.githubusercontent.com/24929411/93283043-af24b000-f80a-11ea-9867-790b7bababe2.png)

정상적으로 pod실행 (kubectl describe pod game-mypage-xxx-xxx)
![image](https://user-images.githubusercontent.com/24929411/93283808-3c1c3900-f80c-11ea-9624-1e89a9b28807.png)

Liveness 설정 변경 (/mnt/aws/healthy --> /tmp/healthy 로 변경)
```
livenessProbe:
  exec:
    command:
    - cat
    - /mnt/aws/healthy --> /tmp/healthy 로 변경 
  failureThreshold: 5
  initialDelaySeconds: 150
  periodSeconds: 5
  successThreshold: 1
  timeoutSeconds: 2
```

파일이 없으니 Liveness 체크에 실패를 했고, pod를 재기동 시킵니다.
![image](https://user-images.githubusercontent.com/24929411/93284147-ff9d0d00-f80c-11ea-8371-aa8684c661ef.png)
