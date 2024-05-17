## 프로젝트 세팅하기

- 프로젝트 폴더 만들고 turbo 설치

```sh
$ npm install -D turbo
```

- package.json 에 workspace 항목 추가하고 "apps" 폴더 생성

```json
{
  "devDependencies": {
    "turbo": "^1.11.3"
  },
  "workspaces": [
      "apps/*"
  ]
}
```

- "apps" 폴더에 nestjs 프로젝트와 react 프로젝트를 각각 'api', 'ui' 로 생성

```sh
$ cd apps
$ npm install -g @nestjs/cli@latest # <== nestjs cli 가 설치되어있지 않다면 실행
$ nest new api
$ npm create vite@latest ui  # <== react와 typescript 선택
```

- 최종 폴더 구조는 apps 폴더 안쪽에 nestjs의 api 프로젝트와 react의 ui 프로젝트가 공존하는 형태이다.
- ![[Pasted image 20240129121445.png]]
- "turbo.json" 파일 생성 [https://turbo.build/repo/docs/reference/configuration](https://turbo.build/repo/docs/reference/configuration)

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "dev": {  /* npm script의 'dev'를 사용하겠다는 뜻  */
        "cache": false
    }
  },
}
```

- 각각의 package.json 파일의 script의 'dev'항목 조정

```json
// api
"scripts": {
    // ...
    "dev": "nest start --watch --preserveWatchOutput", /* "start:dev"되어있었을 것임 */
    // ...
}
```

```json
// ui
"scripts": {
    "dev": "vite",  // <- 이부분 
    "build": "tsc && vite build",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "preview": "vite preview"
  },
```

- 마지막으로 root에 있는 package.json 수정

```json
"scripts" : {
    "dev": "turbo run dev"
}
```

## 로컬개발을 위한 Proxy 연결

- vite.config.ts에 proxy 추가

```ts
export default defineConfig({
  plugins: [react()],
  // 여기서 부터 추가
  server: {
    proxy: {
      "/api": {
        target: "http://localhost:3000",
        changeOrigin: true,
      },
    },
  },
  // 여기까지
});
```

- main.ts에 global prefix 추가

```ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.setGlobalPrefix('api');  //                <-- 이부분
  await app.listen(3000);
}
```

## 빌드하고 운영하기

- turbo.json에 "build"도 추가해보자

```json
"pipeline": {
        "dev": {
            "cache": false
        },
        "build": {
            "dependsOn": [
                "^build"
            ], 
            "outputs": [
                "dist/**"
            ]
        }
    }    
```

- "build" 스크립트도 추가

```json
  "scripts" : {
    "dev": "turbo run dev",
    "build": "turbo run build"
  },
```

- build한 이후 배포직전의 상태라면 빌드된 react만 필요할 것이다. NestJS의 serve static을 이용해보자

```sh
$ npm install --workspace api @nestjs/serve-static
```

- [https://docs.nestjs.com/recipes/serve-static](https://docs.nestjs.com/recipes/serve-static)

```ts
@Module({
  imports: [
    ServeStaticModule.forRoot({
      rootPath: join(__dirname, '../..', 'ui', 'dist'),
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

- 운영환경에서 실행할 "start" 스크립트 도 추가

```json
{
  "scripts" : {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "start": "node apps/api/dist/main"  // <-- 여기
  },
  "devDependencies": {
    "turbo": "^1.11.3"
  },
  "workspaces": ["apps/*"]
}
```