1. Local PC에서 프로젝트를 생성한다.
2. http://github.com/new 에 가서 프로젝트를 생성한다.
3. 생성이 되면 해당 URL을 복사한다.
4. Local PC의 생성된 프로젝트로 가서 
	git remote add orgin <복사하URL을 입력한다.>
	예) git remote add orgin https://github.com/jys5619/hi-nest

## **fatal: not a git repository (or any of the parent directories): .git** 이러한 에러가 발생했습니다!
```null
git init
git remote add origin (GitHub주소)
```

**1. 로컬 저장소의 git 히스토리 삭제**

```
rm -rf .git
```

**2. 로컬 저장소를 다시 초기화**

```
git init
```

**3. 초기화할 파일을 추가 & 커밋**

```
git add . $ git commit -m "first project commit"
```

**4. 커밋 히스토리 확인**

```
git log
```

**5. 저장소 연결 후 푸시**

```
git remote add origin <url> $ git push -u --force origin master
```