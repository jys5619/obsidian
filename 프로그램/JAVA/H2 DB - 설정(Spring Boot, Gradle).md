### build.gradle
	implementation 'org.springframework.boot:spring-boot-starter-jdbc'
    implementation 'com.h2database:h2'

### 
# h2 database web으로 확인
```properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
# spring - h2 연결

spring.datasource.driverClassName=org.h2.Driver
spring.datasource.url=jdbc:h2:file:./db/data;AUTO_SERVER=TRUE
spring.datasource.username=sa
spring.datasource.password=
```

위처럼 하면 파일이 자동 생성됨.
접속 : http://localhost:xxxx/h2-console