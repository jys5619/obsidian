## 오류 : `Delete `␍` eslint (prettier/prettier)`
```
.eslintrc.js 파일에 아래 문구 추가
rules: {
	'prettier/prettier': ['error',{endOfLine: 'auto'}],
}
```