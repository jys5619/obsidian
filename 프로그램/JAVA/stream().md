```java
List<TempVO> tempList = 목록조회...;

// Join
String tempStr = tempList.stream().map(TempVO::getString).collect(Collectors.joining(","));

// Set
Set<String> tempSet = tempList.stream().map(TempVO::getString).collect(Collectors.toSet());

// List
List<String> tempStrList = tempList.stream().map(TempVO::getString).toList();

// Map<String, Object>
Map<String, TempVO> tempMap = tempList.stream().collect(Collectors.toMap(TempVO::getString, vo -> vo));

// Map<String, Map<String, String>>
Map<String, TempVO> tempMap = tempList.stream().collect(Collectors.toMap(TempVO::getString, vo -> Map.of("key1", vo.getVal01(), "key2", vo.getVal02())));

// Map<String, List<TempVO>>
Map<String, List<TempVO>> tempListMap = tempList.stream().collect(Collectors.groupingBy(vo -> vo.getString()));

// Stream의 reduce 이용 Sum
Integer sum1 = numbers.stream().reduce(0, Integer::sum);  

// IntStream의 sum 이용  
int sum2 = numbers.stream().mapToInt(i -> i).sum();
```
