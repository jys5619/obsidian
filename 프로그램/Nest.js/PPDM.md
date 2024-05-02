## Nextjs 설치
```sh
node -v
npm i -g @nestjs/cli
next -v
```

## 프로젝트 생성
```sh
nest new ppdm
cd ppdm
```

#### 설정
##### .eslintrc.js
```sh
rules: {
	'prettier/prettier': ['error',{endOfLine: 'auto'}],
}
```
## Monorepo 프로젝트 구조
```sh
nest g app ppdm-admin
nest g app ppdm-batch
nest g lib ppdm-sqlite-entity
```

#### 파일 삭제 및 수정
* controller, service, spec 파일 삭제
* module.ts, index.ts 파일에서 controller, service삭제로 나는 오류 수정

#### main.ts 파일 Port 수정
* ppdm : 3000
* ppdm-admin: 3001
* ppdm-batch: 3002

## Swagger 설치
```sh
npm install --save @nestjs/swagger swagger-ui-express
```

#### Swagger 설정
```ts
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  const swaggerConfig = new DocumentBuilder()
    .setTitle('PPDM') // PPDM Admin, PPDM Batch
    .setDescription('PPDM CRUD') //
    .setVersion('1.0')
    .addTag('ppdm')
    .build()
  const document = SwaggerModule.createDocument(app, swaggerConfig)
  SwaggerModule.setup('api', app, document)
  
  await app.listen(3000);
}
```

## TypeOrm + Sqlite 설치
```sh
npm i @nestjs/typeorm typeorm typeorm-naming-strategies class-transformer class-validator better-sqlite3 sqlite3
```

#### TypeOrm Config

##### 개발 Sqlite 연결
###### ==**dev-typeorm.config.ts**==: ppdm/libs/ppdm-sqlite-entity/libs/ppdm-sqlite-entity/src\share/config/dev-typeorm.config.ts
```ts
import { TypeOrmModuleOptions } from '@nestjs/typeorm';

export const devTypeOrmConfig: TypeOrmModuleOptions = {
  type: 'better-sqlite3',
  database: 'dev/ppdm.db',
  synchronize: true,
  entities: [__dirname + '/../../entities/**/*.entity{.ts,.js}'], //
  logging: true,
  dropSchema: true,
};
```

###### .gitignore 에서 db폴더는 제외한다.
```
/dev
```
##### CustomerRepository 공통 설정
###### ==**typeorm-ex.decorator.ts**== : ppdm/libs/ppdm-sqlite-entity/src/share/customer-repository/typeorm-ex.decorator.ts
```ts
import { SetMetadata } from '@nestjs/common';

export const TYPEORM_EX_CUSTOM_REPOSITORY = 'TYPEORM_EX_CUSTOM_REPOSITORY';

export function CustomRepository(entity: any): ClassDecorator {
  return SetMetadata(TYPEORM_EX_CUSTOM_REPOSITORY, entity);
}
```

==**typeorm-ex.module.ts**== : ppdm/libs/ppdm-sqlite-entity/src/share/customer-repository/typeorm-ex.module.ts
```ts
import { DynamicModule, Provider } from '@nestjs/common';
import { getDataSourceToken } from '@nestjs/typeorm';
import { DataSource } from 'typeorm';
import { TYPEORM_EX_CUSTOM_REPOSITORY } from './typeorm-ex.decorator';

export class TypeOrmExModule {
  public static forCustomRepository<T extends new (...args: any[]) => any>(
    repositories: T[],
  ): DynamicModule {
    const providers: Provider[] = [];

    for (const repository of repositories) {
      const entity = Reflect.getMetadata(
        TYPEORM_EX_CUSTOM_REPOSITORY,
        repository,
      );

      if (!entity) {
        continue;
      }

      providers.push({
        inject: [getDataSourceToken()],
        provide: repository,
        useFactory: (dataSource: DataSource): typeof repository => {
          const baseRepository = dataSource.getRepository<any>(entity);
          return new repository(
            baseRepository.target,
            baseRepository.manager,
            baseRepository.queryRunner,
          );
        },
      });
    }

    return {
      exports: providers,
      module: TypeOrmExModule,
      providers,
    };
  }
}

```

##### 공통 Entity 생성
##### ==**ppdm-base.entity.ts==: ppdm/libs/ppdm-sqlite-entity/src/share/base-entity/ppdm-base.entity.ts
```ts
import { CreateDateColumn, PrimaryGeneratedColumn } from 'typeorm';

export class PpdmBaseEntity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @CreateDateColumn({ nullable: true, comment: '생성일시' })
  createdAt: Date;

  @CreateDateColumn({ nullable: true, comment: '수정일시' })
  updatedAt: Date;
}
```

##### User Entity, Repository 생성
###### ==**user.entity.ts**==: ppdm/libs/ppdm-sqlite-entity/src/entities/user/user.entity.ts
```ts
import { Column, Entity } from 'typeorm';
import { PpdmBaseEntity } from '../../share/base-entity/ppdm-base.entity';

@Entity({ name: 'user', comment: '사용자' })
export class UserEntity extends PpdmBaseEntity {
  @Column({
    type: 'varchar',
    length: 100,
    nullable: false,
    comment: '사용자명',
  })
  name: string;

  @Column({
    type: 'varchar',
    length: 100,
    nullable: false,
    unique: true,
    comment: '이메일',
  })
  email: string;
}
```

###### ==**user.repository.ts**==: ppdm/libs/ppdm-sqlite-entity/src/entities/user/user.repository.ts
```ts
import { Repository } from 'typeorm';
import { UserEntity } from './user.entity';
import { CustomRepository } from '../../share/customer-repository/typeorm-ex.decorator';

@CustomRepository(UserEntity)
export class UserRepository extends Repository<UserEntity> {}
```

###### ==**user.module.ts**==: ppdm/libs/ppdm-sqlite-entity/src/entities/user/user.module.ts
```ts
import { Module } from '@nestjs/common';
import { UserRepository } from './user.repository';
import { TypeOrmExModule } from '@app/ppdm-sqlite-entity/share/customer-repository/typeorm-ex.module';

@Module({
  imports: [TypeOrmExModule.forCustomRepository([UserRepository])],
  exports: [TypeOrmExModule],
})
export class UserModule {}
```
###### ==**ppdm-sqlite-entity.module.ts**==: D:\dev\workspace\ppdm\libs\ppdm-sqlite-entity\src\ppdm-sqlite-entity.module.ts
```ts
import { Global, Module } from '@nestjs/common';
import { UserModule } from './entities/user/user.module';
import { TypeOrmModule } from '@nestjs/typeorm';
import { devTypeOrmConfig } from './share/config/dev-typeorm.config';

@Global()
@Module({
  imports: [TypeOrmModule.forRoot(devTypeOrmConfig), UserModule],
  exports: [UserModule],
})
export class PpdmSqliteEntityModule {}

```

## Users 생성
```sh
nest g res users
> ppdm
```

#### 파일을 이동한다.

.package.js  start, build 설정 
```json
  "scripts": {
    "build": "nest build ppdm --tsc",
    "build:admin": "nest build ppdm-admin --tsc",
    "build:batch": "nest build ppdm-batch --tsc",
    "start": "nest start ppdm --tsc",
    "start:admin": "nest start ppdm-admin --tsc",
    "start:batch": "nest start ppdm-batch --tsc",
    "start:dev": "nest start ppdm --watch --tsc",
    "start:dev:admin": "nest start ppdm-admin --watch --tsc",
    "start:dev:batch": "nest start ppdm-batch --watch --tsc",
    ...
  }
```
