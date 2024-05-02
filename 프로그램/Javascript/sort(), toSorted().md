- 첫 번째 인자가 두 번째 인자보다 작으면 음수를 반환
- 첫 번째 인자가 두 번째 인자보다 크면 양수를 반환
- 첫 번째 인자가 두 번째 인자와 같으면 `0`을 반환
### 숫자 정렬 sort함수가 예상대로 자동되지 않는 예시
```js
[3, 1, 2].sort();
// [1, 2, 3]

const nums = [3, 1, 2];
const sortedNums = nums.toSorted();
console.log({ nums, sortedNums });

/*
// nums 자체값을 정렬시킴지 않고 값을 복제하여 정렬함.
// nums === sortedNums; // false
{
  nums: [1, 2, 3],
  sortedNums: [1, 2, 3]
}
*/


const nums = [3, 1, 2];
const sortedNums = nums.sort();
console.log({ nums, sortedNums });

[100, 3, 1, 20].sort();
// [1, 100, 20, 3]
```

### 맞는 예시
```js
[-3, 2, 0, 1, 3, -2, -1].sort((a, b) => a - b);
// [-3, -2, -1, 0, 1,  2,  3]

[-3, 2, 0, 1, 3, -2, -1].sort((a, b) => b - a);
// [3, 2, 1, 0, -1, -2, -3]
```

## 객체인 경우

### 틀린 예시
```js
countries.sort(); // '[object Object]`가 되어 배열 내의 모든 객체의 크기가 동일하다고 판단
/**
[
  { no: 1, code: "KR", name: "Korea" },
  { no: 2, code: "CA", name: "Canada" },
  { no: 3, code: "US", name: "United States" },
  { no: 4, code: "GB", name: "United Kingdom" },
  { no: 5, code: "CN", name: "China" },
]
*/
```

### 맞는 예시
```js
countries.sort((a, b) => a.code.localeCompare(b.code));
/**
[
  { no: 2, code: 'CA', name: 'Canada' },
  { no: 5, code: 'CN', name: 'China' },
  { no: 4, code: 'GB', name: 'United Kingdom' },
  { no: 1, code: 'KR', name: 'Korea' },
  { no: 3, code: 'US', name: 'United States' }
]
*/

countries.sort((a, b) => b.no - a.no);
/**
[
  { no: 5, code: "CN", name: "China" },
  { no: 4, code: "GB", name: "United Kingdom" },
  { no: 3, code: "US", name: "United States" },
  { no: 2, code: "CA", name: "Canada" },
  { no: 1, code: "KR", name: "Korea" },
]
*/
```