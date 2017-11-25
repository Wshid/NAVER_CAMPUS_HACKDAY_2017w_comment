NAVER CAMPUS HACKDAY 2017 WINTER - Comment
================

댓글 증감수 기반 콘텐츠 랭킹 시스템 개발
---------

## 목표
현재 가장 인기있는 기사의 최신 댓글 N개을 모아 볼 수 있는 시스템 개발.

## 구현 방법
`
실시간으로 들어오는 데이터를 1분마다 (svc_id, cnts_id)별로 count를 계산하여 타 테이블에 저장한다.
이후 매 분마다 저장된 데이터를 rank테이블로 옮겨 top 100으로 관심도를 확인할 수 있도록 한다.
`

## 구현 로직
1. 실시간으로 cmt_cnts 테이블에 컨텐츠와 댓글이 업로드 된다.
2. cmt_cnts로 부터 (svc_id, cnts_id)를 받아 count를 게산하여 rpcount 테이블에 저장한다.
3. 1분마다 batch 작업을 실시한다.
  - timestamp를 분으로 나눈 값을 String형태로 cur_time_min 속성으로 지정한다.
  - Rank 테이블에 rpcount top 100을 업로드한다.
  - 현재 시간에 사용되지 않는 rpcount 테이블 중 하나를 제거한다.

## 성과
1. 실제 spring framework를 사용하면서 DAO, mybatis, BO등의 로직을 배우게 되었다.
2. mybatis를 사용함으로써 backend단에서 xml로 query를 요청하는 방법을 습득하였다.
3. 실시간으로 들어오는 대량의 데이터를 처리하는 방법에 대해 고민하고 해결하게 되었다.

## 테이블 명세
크게 cmt_cnts, rpcount, rank테이블로 이루어져 있다.
1. cmt_cnts
![Alt text](/src/images/cmt_cnts.png)
2. rpcount
![Alt text](/src/images/rpcount.png)
3. rank
![Alt text](/src/images/rank.png)


## 테이블 설명

### 1) cmt_cnts
- 데몬에 의해 콘텐츠와 댓글이 실시간으로 추가된다.
- (serviceid, contentid)라는 컨텐츠에 comment가 추가되는 방식으로 구현된다.

### 2) rpcount
- cmt_cnts에 실시간으로 댓글이 추가 될 때, 이 테이블에 (svc_id, cnts_id)및 count를 추가한다.
- insertOrUpdateCount 함수를 통하여 해당 키값(svc_id, cnts_id)에 따른 카운트 관리작업을 한다.
  - 해당 키가 없다면 카운트를 1로 초기화 하여 새로운 행을 추가하며, 이미 존재한다면 해당 키에 count+1 작업을 한다.
- 1분 간격의 배치작업을 수행한다.
- 총 5개의 테이블이 존재한다.
  - rpcount_1, rpcount_2, rpcount_3, rpcount_4, rpcount_5
- 매 분마다 다른 테이블에 순차적으로 count를 측정하여 값이 주입된다.
- 다음에 값을 추가하기 위해, DeleteAllCount 함수를 통하여 카운트를 채우는 동안, 다른 테이블의 값을 비워준다.


### 3) rank
- rpcount에서 기록된 값중에 top 100을 이 테이블에 저장한다.
  - SELECT * FROM rpcount_X ORDER BY count DESC LIMIT 100
- cur_time_min이라는 속성을 통하여 매 분마다 top100을 조회할 수 있다.
- rpcount 테이블은 5분 단위로 내용이 새로고침 되는 반면, rank테이블은 꾸준히 유지된다.
