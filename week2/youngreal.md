### SRP
기존엔 이 모듈의 변경 이유가 1개인가? 가 SRP의 기준으로 생각했음
=> 이유를 센다는것 자체가 주관적

코드가 바뀌는 상황을 떠올리고, 그 상황들이 서로 독립적이면 분리해라
  - 조회 화면 바꾸는 상황 ↔ 계산 규칙 바꾸는 상황 → 독립적 → 분리
  - 정산 생성 ↔ 정산 재생성 → 같이 바뀜 → 합쳐도 됨

같은 도메인이면 한 파일에 있는 게 당연하다고 생각했다
예를 들어 정산 관련이면 정산 클래스 하나에 다 넣었을 텐데, 지금 프로젝트 보니까 정산이 3개 파일로 나뉘어 있음
처음엔 "왜 이렇게 나눠놨지?" 싶었는데 책의 Employee 예시(calculatePay, save, Facade)랑 유사한 구조. 같은 도메인이라도 바뀌는 이유가 다르면 분리하는 게 맞음.

### OCP 
protection hierarchy 개념. 
비즈니스 룰이 가장 보호받고 view가 가장 덜보호 받음. 이해한걸로 보자면

  Domain (OrderSettlement)     ← 외부를 아무것도 import 안 함
    ↑
  Application (Service, Executor) ← Domain을 import
    ↑
  Route / API                    ← Application을 import

도메인은 바깥세상을 모르는 상황 

SRP랑 OCP가 따로노는게 아니었다. SRP를 적용하고 DIP로 의존성을 정리하면 OCP가 달성된다고함 (75p)



### LSP에 대해 
거의 변하지 않을 정책들도 별도 DB나 설정파일로 관리해줘야하는걸까? 하드코딩과 실제로 configuration으로 관리대상에 들어가는 녀석들의 기준은? 

### ISP에 대해
  // 좋은 패턴 — index.ts(공개 인터페이스)를 통해 import
  import { CompanyNotFoundError } from '@/server/company';

  // 별로인 패턴 — 내부 파일을 직접 import
  import type { Company } from '@/server/company/domain/models/company.entity';
  
두 번째 방식은 company 모듈의 내부 구조에 직접 의존하는 거라, company.entity.ts 파일명이나 경로가 바뀌면 다 깨짐. 책의 S → F → D 예시랑 같음. 안 쓰는 것까지 끌고들어온 건 아니지만, 내부 구조에 커플링된 거. index.ts를 통했으면 내부를 자유롭게 리팩토링할 수 있었을 텐데.


  만약 index.ts 없이 외부에서 이렇게 쓴다고 해보면:
  // Route에서 직접 내부 파일을 import
  ```java
  import { SettlementService } from '@/server/order-settlement/application/commands/settlement.service';
  import { SettlementExecutor } from '@/server/order-settlement/application/commands/settlement-executor.service';
  import { SettlementRepository } from '@/server/order-settlement/infrastructure/settlement.repository';
  ```
  
  이러면 Route가 모듈 내부 구조 전체를 알아야 함. settlement-executor.service.ts 파일명을 바꾸거나 폴더를 정리하면? Route가 깨짐. Route는 Executor를 직접 쓸 일도
  없는데.


  // index.ts — 외부에 필요한 것만 공개
  export { SettlementService } from './application/commands/settlement.service';
  export type { CreateSettlementRequest } from './application/commands/dto/...';
  export type { SettlementResponse } from './application/commands/dto/...';
  // SettlementExecutor는 여기 없음 → 외부에서 못 봄
  // SettlementRepository도 여기 없음 → 외부에서 못 봄

  그래서 Route에서는:

  // 내부 구조 몰라도 됨. index.ts가 공개한 것만 씀.
  import { SettlementService } from '@/server/order-settlement';

  이제 내부에서 SettlementExecutor를 리팩토링하든, 파일을 쪼개든, 폴더를 옮기든 — Route는 전혀 모르고 영향도 안 받음. 이게 책에서 말하는 "User1한테 op1만 보이는
  인터페이스를 만들어라"와 같은 거예요.


### DIP에 대해 
String 같은 건 구체 클래스여도 의존해도 된다." 이 말. 모든 걸 인터페이스로 감싸는 게 아니라, 자주 바뀌는 것만 조심하면 됨
구체화에 아예 의존하지 말라는 의미인줄알았음.

### SOLID 전체를 읽고 나서 — 전반적인 레슨     
관련내용을 ai규칙에 넣어주고 좀더 강하게 리뷰 받아보는것도 재밌을듯.
  레슨  "원칙을 몰라도 좋은 코드는 원칙을 지키고 있더라"

  "String 같은 건 구체 클래스여도 의존해도 된다." 모든 걸 인터페이스로 감싸는 게 아니라, 자주 바뀌는 것만 조심하면 됨. 
  완벽한 DIP는 아니지만 테스트할 수 있으면 충분함. 원칙을 100% 지키는 게 목적이 아니라, 원칙이 해결하려는
  문제를 이해하는 게 목적이라는 걸 배움.
