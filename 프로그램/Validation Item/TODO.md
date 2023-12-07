- ValidationItem 목록 추출 ALL 처리 방법 확인 필요
	- 배치 ValidationItem 조회
	- 화면 등록 배치 처리
- bateDate 설정 운영에 맞게 설정
- Error Monitoring 결과 값 저장 로직 개발
- Error Monitoring Validation Item 각각 수정
- 배치 완료 작업 결과를 저장 하여 메일 보내는 데서 실행 할 수 있도록 하는 연계 작업 필요
---
- obid단건일 경우 modelType, modelForm 맞는지 확인 필요
- Model 여러개 선택해서 실행하는 화면 개발 필요

- ~~Forbidden 오류 엑셀 등록오류 수정~~ : 2023-11-02
- ~~BOM Compare 임시 파트/모델 조회 되도록 수정~~ : 2023-11-02
- ~~레포트 Spec 길면 문자열 가리고 tooltip 처리~~ : 2023-11-03
- ~~HA타이틀 변경~~ : 2023-11-03

- ~~Change 조회안된는 오류 수정 - CodeDetail쪽 오류임~~ : 2023-11-06
- ~~BOM Compare QTY 비교 수정~~ : 2023-11-06
- 레포트 Change List 엑셀 다운로드


[HA]
ECO Validation Management 데이터 생성시 Code다건 선댁되게 수정함.
BOM/UIT Error Mail Receiver 사이트 전체 출력되게 수정함. 데이터가 안맞는건 빨간색 Row로 표시함.
(HE)Model BOM Status Report 속도향상을 위해 반복 쿼리문 수정