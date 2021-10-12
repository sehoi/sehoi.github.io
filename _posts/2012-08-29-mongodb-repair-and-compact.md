---
layout: single
classes: wide
title: "[MongoDB] repair Database & compact Command"
date: 2012-08-29
description: "NoSQL Database인 MongoDB에서 사용 가능한 디스크 용량 확보하기 위한 두 가지 방법에 대해서 정리한 글입니다. 두 가지 방법은 각각 db.repairDatabase() 명령과 collection 별로 compact 명령을 실행하는 것입니다."
categories:
  - Database
tags: [database, mongodb, nosql, repair database]
comments: true
share: true
---

MongoDB에서 데이터를 삭제하더라도 실제 사용 가능한 디스크 용량은 그대로인데,
그 이유는 MongoDB에서 디스크를 미리 할당해서 사용하는 방식 때문이다.([참고](http://www.mongodb.org/display/DOCS/Excessive+Disk+Space))
<br />

#### MongoDB에서 사용 가능한 디스크 용량을 확보하기 위한 두가지 방법
1. 특정 DB에 대해 db.repairDatabase() 명령 실행 ([참고](http://www.mongodb.org/display/DOCS/Durability+and+Repair))
  - 장점: 실제 물리적인 디스크 용량이 늘어남
  - 단점: 실행 시간이 길며 lock이 걸림, DB 크기만큼의 여유 디스크 용량을 필요로 함
    - 따라서 디스크 용량이 충분하지 않은 경우엔 실행할 수 없다
  <br />
2. 각 collection마다 compact 명령 실행 ([참고](http://www.mongodb.org/display/DOCS/compact+Command))
  - 장점: collection 대상으로 실행하기 때문에 repairDatabase보다는 훨씬 적은 디스크 용량을 사용
  - 단점: 실행 후 실제 디스크 용량은 늘지 않고, 이미 할당된 공간의 파편화를 정리하여 사용 가능하게 만들어 줌
    - 따라서 디스크 용량이 충분하지 않은 경우엔 실행할 수 없다
  <br />
<br /><br />


#### 데이터 증가를 효율적으로 관리하는 방법
1. capped collection
  - collection 생성 시 최대 크기와 document 개수를 옵션으로 설정 가능 ([참고](http://www.mongodb.org/display/DOCS/Capped+Collections))
  - 장점: collection 최대 크기까지 도달 시 자동으로 오래된 데이터를 삭제 후 신규 데이터를 삽입함
  - 단점: delete는 불가능, update는 이전 doc 크기와 동일한 경우만 가능
  <br />
2. mongod --smallfiles 옵션 사용
  - 파일 할당 크기를 기본 64MB~2GB에서 16MB~512MB로 설정([참고](http://www.mongodb.org/display/DOCS/Command+Line+Parameters))
  - 장점: 들어오는 데이터 크기가 작을 경우 DB 크기 증가 속도 감소
  - 단점: 파일 할당이 빈번해져서 속도가 느려질 수 있음
<br /><br />

#### serverside-JS
- [참고](http://stackoverflow.com/questions/4555938/auto-compact-the-deleted-space-in-mongodb/4560096#4560096)
