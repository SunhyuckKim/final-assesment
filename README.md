# final-assesment

구현 Repository

<img width="400" alt="스크린샷 2020-09-17 오후 1 24 48" src="https://user-images.githubusercontent.com/68723566/93420243-86271c80-f8e9-11ea-9a2e-88279282273d.png">
<img width="400" alt="스크린샷 2020-09-17 오후 1 25 30" src="https://user-images.githubusercontent.com/68723566/93420251-87f0e000-f8e9-11ea-8db5-ed01ce595944.png">

## 개인 AWS계정에 팀과제 및 개인과제 추가된 내용이 EKS 배포 됨을 확인
<img width="800" alt="스크린샷 2020-09-17 오전 11 17 32" src="https://user-images.githubusercontent.com/68723566/93420710-7cea7f80-f8ea-11ea-891e-b8b1a4b2d2fe.png">


### 이벤트 스토밍

## 팀과제 완성된 모형
![image](https://user-images.githubusercontent.com/68723566/93046088-ecb2fd00-f693-11ea-836f-bd166b106df1.png)

## 개인과제 추가된 모형
고객이 받은 리워드로 교환한 gift에 대한 내역을 특정 메신저(텔레그램)를 통해 전달하는 기능 추가

# 시나리오 
- Mission 달성 시 Reward를 지급 받아 Mission Table이 수정되면 Telegramdmfh 이벤트를 전달한다.
- 새로운 Application인 Telegram은 전달받은 이벤트를 통해 사용자에게 메시지를 전달해준다.
- 메시지 전송이 완료되면 다시 Mission시스템이 이벤트를 통해 알려준다
- gift를 사용하면 텔레그램 메신저에 Request/Response 로 보내준다
![telegram](https://user-images.githubusercontent.com/68723566/93432246-343dc100-f900-11ea-8627-978191ac2670.png)

변경된 소스코드
- Mission 서비스에 MissionUpdated.java, MessageUpdated.java 추가
```
package game;

public class MissionUpdated  extends AbstractEvent{
    private Long id;
    private String status;
    private Long rewardId;
    private Long customerId;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }

    public Long getRewardId() {
        return rewardId;
    }

    public void setRewardId(Long rewardId) {
        this.rewardId = rewardId;
    }

    public Long getCustomerId() {
        return customerId;
    }

    public void setCustomerId(Long customerId) {
        this.customerId = customerId;
    }
}


package game;

public class MessageUpdated extends AbstractEvent{
    private Long id;
    private Long missionId;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }
    public Long getMissionId() {
        return missionId;
    }

    public void setMissionId(Long missionId) {
        this.missionId = missionId;
    }
}

```
- Mission 서비스에 PostUpdate코드 추가(Mission.java) 
```
    @PostUpdate
    public void onPostUpdate(){
        MissionUpdated missionUpdated = new MissionUpdated();
//        BeanUtils.copyProperties(this, missionUpdated);
        missionUpdated.setCustomerId(this.customerId);
        missionUpdated.setId(this.id);
        missionUpdated.setRewardId(this.rewardId);
        missionUpdated.setStatus(this.status);
        missionUpdated.publishAfterCommit();
   }
```
- Mission PolicyHandler.java 코드 추가
```
    @StreamListener(KafkaProcessor.INPUT)
    public void wheneverMessageUpdated_SendMessage(@Payload MessageUpdated messageUpdated){

        // 메시지 업데이트 확인로직
        //
        if(messageUpdated.isMe()){
            System.out.println("##### listener SendMessage : " + messageUpdated.toJson());
        }
    }
```

-  PolicyHandler.java에 텔레그램 
```
package game;

import game.config.kafka.KafkaProcessor;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.stereotype.Service;

@Service
public class PolicyHandler{
    @Autowired
    TelegramRepository telegramRepository;
    @StreamListener(KafkaProcessor.INPUT)
    public void onStringEventListener(@Payload String eventString){

    }

    @StreamListener(KafkaProcessor.INPUT)
    public void wheneverMissionUpdated_Alert(@Payload MissionUpdated missionUpdated){


        if(missionUpdated.isMe()){
            telegram telegram = new Telegram();
            telegram.setMissionId(missionUpdated.getId());
            telegram.setStatus(missionUpdated.getStatus());

            telegramRepository.save(telegram);
            ### 텔레그램 메신저를 보내는 로직 ###
            #                         
            #
            System.out.println("##### listener Alert : " + missionUpdated.toJson());
        }
    }

}

```
- Gift 에서 telegram 서비스로 Request/Response 로직 추가 (TelegramService.java)
```
package game.external;


import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@FeignClient(name="telegram", url="${api.url.telegram}", fallback=telegramServiceFallback.class)
public interface KakaoTalkService {
    @RequestMapping(method= RequestMethod.GET, path="/telegram")
    public void use(@RequestBody telegram kakaoTalk);
}

```
- Mypage 에서 telegram 관련 코드 추가 


## Saga

시나리오: Mission -> reward -> Mission Update -> telegram -> Message update 이벤트 -> Mission

![telegram](https://user-images.githubusercontent.com/68723566/93428011-baa2d480-f8f9-11ea-92f7-0203bdcae15c.png)
![telegram](https://user-images.githubusercontent.com/68723566/93428006-b8d91100-f8f9-11ea-957e-68acdfc45204.png)
![telegram](https://user-images.githubusercontent.com/68723566/93427997-b676b700-f8f9-11ea-84a6-454b5d692b4c.png)


## CQRS

Database 조회 업무만을 수행하기 위한 mypage 개발
1. 소스
```
  profiles: docker
  datasource:
    url: jdbc:mariadb://${DB_URL}/admin06_mariadb?useUnicode=yes&characterEncoding=UTF-8
    driver-class-name: org.mariadb.jdbc.Driver
    username: ${DB_USER}
    password: ${DB_PASSWORD}
```
```
    @StreamListener(KafkaProcessor.INPUT)
    public void whenExchanged_then_UPDATE_Used(@Payload Used used) {
        try {
            if (used.isMe()) {
                // view 객체 조회
                List<Mypage> mypageList = mypageRepository.findByRewardId(used.getId());
                for(Mypage mypage : mypageList){
                    // view 객체에 이벤트의 eventDirectValue 를 set 함
                    mypage.setTelegramStatus(used.getStatus());
                    // view 레파지 토리에 save
                    mypageRepository.save(mypage);
                }
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }
```
![telegram](https://user-images.githubusercontent.com/68723566/93428557-a6aba280-f8fa-11ea-8563-524a8f51d955.png)



## 동기식 호출 과 Fallback 처리

gift를 사용하면 Telegram으로 Request/Response됨 

- Telegram 서비스를 호출하기 위하여 Stub과 (FeignClient) 를 이용하여 Service 대행 인터페이스 (Proxy) 를 구현 

gift 시스템 TelegramService.java
```
@FeignClient(name="telegram", url="${api.url.telegram}", fallback=TelegramServiceFallback.class)
public interface TelegramService {
    @RequestMapping(method= RequestMethod.GET, path="/telegram")
    public void use(@RequestBody Telegram telegram);
}
```

```
# Gift.java (Entity)

    @PostPersist
    public void onPostPersist(){

        game.external.telegram telegram = new game.external.telegram();
        // mappings goes here
        telegram.setId(this.getId());
        telegram.setStatus("send message!!!!");
        GiftApplication.applicationContext.getBean(game.external.TelegramService.class)
                .use(telegram);
    }
```



### 게이트웨이


#istio Virtual Service에 telegram path 추가함
![path](https://user-images.githubusercontent.com/68723566/93420493-fdf54700-f8e9-11ea-9968-9195e4b00d6f.png)

#설치된 istio확인
![istio](https://user-images.githubusercontent.com/68723566/93420501-0188ce00-f8ea-11ea-8dd7-4159ae11892b.png)


#적용된 yml 확인
![yml](https://user-images.githubusercontent.com/68723566/93420504-03eb2800-f8ea-11ea-99c1-c25004d7a83e.png)


## CI/CD 적용

AWS Codebuild를 통한 webhook 빌드 기능 적용
build script는 각 프로젝트 루트 경로에 buildspec.yml에 포함함
![cicd](https://user-images.githubusercontent.com/68723566/93420805-b28f6880-f8ea-11ea-9888-6739b9c864f2.png)


## Autoscale out 적용
test중 

## ConfigMap/Persistence Volume 적용


Java application.yml 환경변수 적용을 위해 ConfigMap 설정

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


pvc 적용
![pvc](https://user-images.githubusercontent.com/68723566/93421409-3c8c0100-f8ec-11ea-8d6b-880fda99e3f6.png)

![pvc](https://user-images.githubusercontent.com/68723566/93423025-fa64be80-f8ef-11ea-8b4a-9e171e4dbd46.png)
![pvc](https://user-images.githubusercontent.com/68723566/93423039-fe90dc00-f8ef-11ea-829e-dbe7e248c889.png)

## Polyglot 적용

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


## Liveness Probe 적용

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
