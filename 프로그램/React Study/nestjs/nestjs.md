### Nest.js

[1월 18, 2024](https://ohhoonim.blogspot.com/2024/01/nestjs.html "permanent link")

## 유튜브 알고리즘이 Nestjs를 추천해왔다

- Dave Gray : [https://youtu.be/8_X0nSrzrCw?si=FqG7DpPo838TTBhY
    
    - prisma
    
    ](https://youtu.be/8_X0nSrzrCw?si=FqG7DpPo838TTBhY%3Cul%3E%3Cli%3Eprisma%3C/li%3E%3C/ul%3E)
- 우아한테크 : [https://youtu.be/Z0d7ZrxY-i0?feature=shared
    
    - @nestjs-library/crud
    
    ](https://youtu.be/Z0d7ZrxY-i0?feature=shared%3Cul%3E%3Cli%3E@nestjs-library/crud%3C/li%3E%3C/ul%3E)
- Marius Espejo : [https://youtu.be/WZtHM4Ph-K8?si=X29Tcj4BvT7GwhEG
    
    - @nestjsx/crud
    
    ](https://youtu.be/WZtHM4Ph-K8?si=X29Tcj4BvT7GwhEG%3Cul%3E%3Cli%3E@nestjsx/crud%3C/li%3E%3C/ul%3E)

## 따라해보다

```sh
$ node -v
$ npm i -g @nestjs/cli # 설치안되어있으면 설치
$ nest -v
$ nest new [프로젝트명]
$ cd [프로젝트명]
$ nest g moudle post
$ nest g controller post
$ nest g service post
$ npm install --save @nestjs/swagger swagger-ui-express
$ npm i --save @nestjs/typeorm typeorm sqlite3
$ npm i @nestjsx/crud class-transformer class-validator
$ npm i @nestjsx/crud-typeorm @nestjs/typeorm typeorm 
```

## 코드추가

- swagger

```ts
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const swaggerConfig = new DocumentBuilder()
    .setTitle('Posts example')
    .setDescription('Posts crud')
    .setVersion('1.0')
    .addTag('posts')
    .build()
  const document = SwaggerModule.createDocument(app, swaggerConfig)
  SwaggerModule.setup('api', app, document)
  await app.listen(3000);
}
```

- typeorm

```ts
// app.module.ts
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'sqlite', 
      database: ':memory:',
      entities: ['dist/**/*.entity{.ts,.js}'],
      dropSchema: true,
      synchronize: true,
    }),
    PostsModule],
})
export class AppModule {}
```

- entity

```ts
// entity.ts
@Entity({name: "post"})
export class Post {
    @PrimaryGeneratedColumn()
    id: number

    @ApiProperty()
    @Column()
    author: string

    @ApiProperty()
    @Column()
    title: string

    @ApiProperty()
    @Column()
    contents: string

    @ApiProperty()
    @Column()
    createDate: string
}
```

- typrorm feature

```ts
// posts.module.ts
@Module({
  imports: [
    TypeOrmModule.forFeature([Post])
  ],
  controllers: [PostsController],
  providers: [PostsService]
})
export class PostsModule {}
```

- service

```ts
// service.ts
@Injectable()
export class PostsService extends TypeOrmCrudService{
    constructor(@InjectRepository(Post) repo) {
        super(repo)
    }
}
```

- constroller

```ts
@Crud({
    model: {
        type: Post
    }
})
@Controller('posts')
export class PostsController implements CrudController{
    constructor(public service: PostsService){}
}
```

## 영상대로 안되다

- 왜 안되나 살펴보니 nestjs가 업데이트 되면서 controller에 파라미터를 주입하는 방법이 바뀐것 같다.
- @nestjsx/crud 깃헙 가보니 개발 끊긴지 오래. 되게끔 일딴 해보자
- 아래처럼하니 일단 되긴 한다.

```ts
@Crud({
    model: {
        type: Post
    }
})
@Controller('posts')
export class PostsController implements CrudController{
    constructor(public service: PostsService){}

    @Override()
    getOne(@Req() req: Request): Promise {
        return this.service.getOne(req['NESTJSX_PARSED_CRUD_REQUEST_KEY'])
    }

    @Override()
    createOne(@Req() req: Request): Promise {
        console.log(req.body)
        return this.service.createOne(req['NESTJSX_PARSED_CRUD_REQUEST_KEY'], req.body)
        // console.log(request)
        // override안해주면 typeorm-crud.service.js 48line에서 에러남.
        // req 변수가 null req
        // return Promise.resolve({
        //     id: 1,
        //     author: "matthew",
        //     title: "title",
        //     contents: "",
        //     createDate: "2024-01-06T22:10:25"
        // })
    }
}
```

## crud와 비교

- 갑자기 우아한테크 채널에서 본 영상이 기억이 나서 코드를 분석해봤다.
    
- 둘다 @Controller에 [@Crud](https://github.com/Crud)() 데코레이션을 추가해주면 지정된 메서드를 만들어준다. 여기서부터 시작
    
- @nestjsx/crud
    
    - ![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj9RZ49ZrnRZ3MAyECWcotDq_a3K47NYUuLrReHdcLAs2XMajylIfo5TkSa0Ni5wh_RhyeryWRur7Ob43grX_OZur0M56yP2M1sN4myxAiIpHhojU-BwtTLyOPfwhf6OL8TUFkUlkkM204uL6736eRj0BN673ZtI7jKglZi8rExcNEMSWoykGyENc7HaDs/s320/Pasted%20image%2020240107093023.png)
- @nestjs-library/crud
    - ![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEipNfOIOsMovIoBp99lv63XDkGRLiVms1d_d4Sv7C0tpTVxvANExXZJKS0dmiTIaFMEF6DM4m_PJh3UK8gw_h9SQOfedO2pXSjsvG2YsDA9na8vQujddtjf0_B7ZVZkx9pup5vrd8vMluwXUgHscdiJH0EqbHtjFuwE00mkANpvaoxN0VMxf5tH7oTTPks/s320/Pasted%20image%2020240107093238.png)
- 예상대로 파라미터 주입하는 부분이 다르다.
    

## 마무리

- nestjs 진영의 분위기를 살펴보니 스프링 진영의 딱 10년전 모습을 보는 것 같다.
- nestjs 어렵지 않으니 스킬하나 추가

[crud](https://ohhoonim.blogspot.com/search/label/crud) [NestJS](https://ohhoonim.blogspot.com/search/label/NestJS)