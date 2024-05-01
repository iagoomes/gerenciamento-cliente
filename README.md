# gerenciamento-cliente

Estou criando esse projeto para abordar bibliotecas e plugins atreladas ao codegen com o contrato openapi.
Lendo alguns dos principais foruns de Java disponíveis na internet percebi que está tendo uma grande mudanças no que se diz respespeito a documento de APIs. 
Com o surgimento do OpenApi Specifacation nas versões 3+ temos a possibilidade de criar um contrato da nossa API antes mesmo de desenvolve-la ou seja estamos defindo como deve ser construida a API.

## As grandes vantagens
### Documentação
Com o contrato da nossa API no padrão Openapi Specifacation podemos utilizar o swagger que consegue interpretar esse contrato e gerar nossa documentação de forma automatica :sunglasses:

### Geração de código
Através da contrato podemos utilizar o [plugin da openapi](https://openapi-generator.tech/) que faz a criação dos nossos endpoints e dos nossos DTOs de forma automática.



## Coda Comigo
#### Framework & Bibliotecas
1. [Spring Boot 3+](https://spring.io/projects/spring-boot)
2. [Maven](https://maven.apache.org/)
3. [Java 21](https://www.oracle.com/br/java/technologies/downloads/#java21)
4. [Lombook](https://projectlombok.org/)
5. [MapStruct](https://mapstruct.org/)
6. [Openapi](https://spec.openapis.org/oas/latest.html#openapi-document)


#### pom.xml inicial

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.2.5</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>br.com.iagoomes</groupId>
	<artifactId>gerenciamento-cliente</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>gerenciamento-cliente</name>
	<description>gerenciamento-cliente</description>
	<properties>
		<java.version>17</java.version>
		<org.mapstruct.version>1.5.5.Final</org.mapstruct.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.mapstruct</groupId>
			<artifactId>mapstruct</artifactId>
			<version>${org.mapstruct.version}</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<excludes>
						<exclude>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok</artifactId>
						</exclude>
					</excludes>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.8.1</version>
				<configuration>
					<source>${java.version}</source> <!-- depending on your project -->
					<target>${java.version}</target> <!-- depending on your project -->
					<annotationProcessorPaths>
						<path>
							<groupId>org.mapstruct</groupId>
							<artifactId>mapstruct-processor</artifactId>
							<version>${org.mapstruct.version}</version>
						</path>
						<!-- other annotation processors -->
					</annotationProcessorPaths>
				</configuration>
			</plugin>
		</plugins>
	</build>

</project>
```

#### Criando nosso contrato
Para criar nosso contrato, vamos utilizar o [Swagger Editor](https://swagger.io/tools/swagger-editor/).
Com ele conseguimos criar e já validar o swagger gerado :sunglasses:.

Nosso contrato será no padrão YAML, tipo de arquivo recomendado pela openapi.
Para mais informações verificar [documentação de especificação](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.3.md#info-object)

Vamos iniciar nosso contrato com as tags obrigatórias
```yaml
openapi: 3.0.3
info:
  title: Gerenciamento de Clientes
  description: API de gerenciamento de Clientes
  version: 1.0.0
servers:
  - url: app-iagoomes/v1
    description: Servidor de desenvolvimento
tags:
  - name: cliente
paths:
  /clientes:
    get:
      tags:
        - cliente
      summary: Lista todos os clientes
      responses:
        '200':
          description: Lista de Clientes
```

O contrato acima aborda apenas as tags principais, a fim de exemplificar o scopo inicial seguindo boas praticas.
Vamos implementar o método POST e deixar o GET de canto por enquanto.

Optei por implemetar o verbo POST agora, porque vamos criar DTOs de entrada e saida com isso conseguimos aproveitar esses DTOs em outros verbos HTTP.
```yaml
paths:
  /clientes:
    post:
      tags:
        - cliente
      summary: Cadastrar cliente
      operationId: criaCliente
      requestBody:
        description: Dados iniciais do cliente
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ClienteRequest'
      responses:
        201:
         description: Account not found
          content:
            application/json:
              schema:
```

Veja que a estrutura acima é muito similar a estrutura que já criamos para o verbo GET.
Incluimos duas tags novas **operationId** e **requestBody**, a nivel de estruturamente do contrato o operationId não é tão importante porem quando os codigos forem gerados ele será o nome do nosso método na nossa controller, já o resquestBody será nosso objeto de entrada passado pelo usuario.

#### $ref
Essa sintaxe é utilizada para acessar definições do nosso proprio contrato, exemplo: **$ref: '#/components/schemas/ClienteRequest'**.
Nese exemplo estamos acessando um objeto criado em **/components/schemas** que por sua vez, esse objeto foi criado para representar nosso DTO de entrada. O mesmo vale para nosso DTO de resposta.

