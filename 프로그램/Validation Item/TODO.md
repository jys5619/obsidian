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


indent-rainbow - 들여쓰기 색입히기
Image preview - 이미지 미리보기
GitHub Theme - VS Code 테마 변경
Material Icon Theme - 폴더 아이콘 표시
Formatting - 코드를 예쁘게 정렬
   - Setting에서 format으로 조회하여  Editor: Default Formatter 에서 Prettier - Code formatter 선택
   - Editor : Format On Save 도 체크
Live Server - html 실시간 반영
👍Setting Sync - Setting 정보를 Github에 저장 하여 그대로 다른 컴퓨터에서 사용가능
  - 업로드 : CTRL + SHIFT + U
  - 다운로드 : CTRL + SHIFT + D
gitlens : Git에 누가 언제 수정했는지 소스에서 확인
Multiple cursor case preserve : 텍스트를 멀티 선택 한 후 변경하면 대소문자를 알아서 변경하여줌

단축키
ALT + SHIFT + I
ALT + SHIFT + Drag
CTRL + Home/End


var gridOptions = {
  columnDefs: [
    { headerName: "Cell", field: "cellValue", cellStyle: getCellStyle }
  ],
  rowData: [
    { cellValue: "A01,B01" }
  ]
};

function getCellStyle(params) {
  // 셀의 값을 쉼표를 기준으로 분리합니다.
  var values = params.value.split(',');
  
  // 각 값에 따라 다른 색상을 지정합니다.
  var colors = values.map(value => {
    if (value === "A01") return 'red';
    else if (value === "B01") return 'blue';
    else return 'black'; // 다른 값은 검은색으로 설정합니다.
  });
  
  // CSS 스타일 문자열을 반환합니다.
  return {
    color: colors.join(', ')
  };
}

// 그리드 생성
var gridDiv = document.querySelector('#myGrid');
new agGrid.Grid(gridDiv, gridOptions);
