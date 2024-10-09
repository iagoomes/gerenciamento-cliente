# Gerenciamento Cliente - Openapi Generator

Estou criando esse projeto para abordar bibliotecas e plugins atreladas ao codegen com o contrato openapi.
Lendo alguns dos principais foruns de Java disponíveis na internet percebi que está tendo uma grande mudanças no que se diz respespeito a documento de APIs. 
Com o surgimento do OpenApi Specifacation nas versões 3+ temos a possibilidade de criar um contrato da nossa API antes mesmo de desenvolve-la ou seja estamos defindo como deve ser construida a API.

## As grandes vantagens
### Documentação
Com o contrato da nossa API no padrão Openapi Specifacation podemos utilizar o swagger que consegue interpretar esse contrato e gerar nossa documentação de forma automatica :sunglasses:

### Geração de código
Através do contrato podemos utilizar o [plugin da openapi](https://openapi-generator.tech/) que faz a criação dos nossos endpoints e dos nossos DTOs de forma automática.



## Coda Comigo
### Framework & Bibliotecas
1. [Spring Boot 3+](https://spring.io/projects/spring-boot)
2. [Maven](https://maven.apache.org/)
3. [Java 21](https://www.oracle.com/br/java/technologies/downloads/#java21)
4. [Lombook](https://projectlombok.org/)
5. [MapStruct](https://mapstruct.org/)
6. [Openapi](https://spec.openapis.org/oas/latest.html#openapi-document)


### pom.xml inicial

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

# Criando seu contrato OpenAPI (Swagger)

Vamos começar criando nosso contrato utilizando o Swagger Editor, uma ferramenta visual que facilita a criação e validação de APIs no padrão OpenAPI (ou Swagger). A partir dela, escreveremos nosso contrato em YAML, o formato mais usado e recomendado pela OpenAPI.

### Estrutura básica do contrato

Vamos criar uma estrutura inicial do contrato com as informações básicas que toda API precisa. Essa estrutura define como a API será documentada e onde ela está hospedada.

```yaml
openapi: 3.0.3
info:
  title: Gerenciamento de Clientes
  description: API para gerenciar clientes
  version: 1.0.0
servers:
  - url: gerenciamento_clientes
    description: Servidor local de desenvolvimento
tags:
  - name: cliente
    description: Endpoints relacionados a clientes
paths:
  /clientes:
    get:
      tags:
        - cliente
      summary: Lista todos os clientes
      responses:
        '200':
          description: Retorna a lista de clientes cadastrados
```

**O que isso significa?**
- openapi: 3.0.3: Estamos usando a versão 3.0.3 da especificação OpenAPI.
- info: Informações sobre nossa API, como o nome, a descrição e a versão.
- servers: Context Path da API.
- tags: Agrupamento de endpoints (pontos de acesso da API) relacionados a clientes, facilitando a organização.
- paths: Define os endpoints da API. Aqui temos o endpoint /clientes, que responde a requisições GET e lista todos os clientes.

---

### Adicionando o método POST

Agora que já temos um endpoint para listar clientes (GET), vamos adicionar um endpoint para criar novos clientes (POST). Esse método vai receber um request body (os dados do cliente) e retornar um response body (a confirmação da criação).

```yaml
paths:
  /clientes:
    post:
      tags:
        - cliente
      summary: Cadastrar um novo cliente
      operationId: criaCliente
      requestBody:
        description: Dados necessários para criar um cliente
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CriaCliente'
      responses:
        '201':
          description: Cliente criado com sucesso
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ClienteCriado'
        '400':
          description: Dados inválidos
```

**O que foi adicionado?**
- post: Estamos definindo que, ao acessar o endpoint /clientes com o método POST, será possível cadastrar um novo cliente.
- operationId: Esse é o nome da operação. Ele é importante quando for gerado o código da API, pois será o nome do método correspondente no código da controller.
- requestBody:
	- description: Uma breve descrição do que esse corpo de requisição representa.
	- required: Define se o corpo da requisição é obrigatório. Aqui, estamos dizendo que sim.
	- content: Define o tipo de dado que o endpoint vai aceitar. No caso, estamos aceitando application/json, ou seja, o corpo da requisição será em formato JSON.
	- schema: Usamos o $ref para referenciar um esquema que vamos criar em /components/schemas/CriaCliente.
- responses: Aqui, estamos definindo o que a API vai retornar após a execução do método POST:
	- 201: Status HTTP que indica que o cliente foi criado com sucesso. Vamos retornar um objeto que segue o esquema ClienteCriado, que vamos definir em breve.
	- 400: Status HTTP que indica que houve algum erro de validação nos dados enviados.

---

### Definindo o Request Body (CriaCliente)

Agora que já referenciamos o objeto CriaCliente no método POST, precisamos definir como será esse objeto. Ele será utilizado como o corpo da requisição quando alguém quiser criar um cliente.

```yaml
components:
  schemas:
    CriaCliente:
      type: object
      properties:
        nome:
          type: string
          description: Nome completo do cliente
        idade:
          type: integer
          description: Idade do cliente
        cpf:
          type: string
          description: CPF do cliente
          pattern: '^\d{3}\.\d{3}\.\d{3}-\d{2}$'
      required:
        - nome
        - idade
        - cpf
      example:
        nome: "Ana Silva"
        idade: 28
        cpf: "123.456.789-00"
```

**Explicando o YAML**

- type: object: Define que CriaCliente é um objeto (com várias propriedades dentro dele).
- properties: Aqui definimos os campos que o objeto vai ter:
	- nome: O nome completo do cliente. O tipo de dado é string.
	- idade: A idade do cliente. O tipo de dado é integer (número inteiro).
	- cpf: O CPF do cliente. O tipo de dado é string, e incluímos uma validação de formato com pattern, que segue o padrão de CPF brasileiro (XXX.XXX.XXX-XX).
- required: Aqui listamos os campos que são obrigatórios no request. No nosso exemplo, os campos nome, idade e cpf são obrigatórios.
- example: Um exemplo de como os dados devem ser enviados. Isso é útil tanto para quem está implementando o frontend quanto para gerar a documentação da API.

---

### Definindo o Response Body (ClienteCriado)

Depois que um cliente for criado com sucesso, vamos retornar um objeto de resposta que contém as informações do cliente, incluindo o ID gerado.

```yaml
components:
  schemas:
    ClienteCriado:
      type: object
      properties:
        id:
          type: integer
          description: ID único do cliente
        nome:
          type: string
          description: Nome completo do cliente
        idade:
          type: integer
          description: Idade do cliente
        cpf:
          type: string
          description: CPF do cliente
          pattern: '^\d{3}\.\d{3}\.\d{3}-\d{2}$'
      example:
        id: 101
        nome: "Ana Silva"
        idade: 28
        cpf: "123.456.789-00"
```

**Explicando o YAML**
- type: object: Define que ClienteCriado é um objeto.
- properties: Campos que compõem o objeto de resposta:
	- id: O identificador único do cliente criado, gerado automaticamente.
	- nome, idade, cpf: São os mesmos campos que recebemos no request, retornados como confirmação.
- example: Um exemplo de resposta que a API retorna após a criação de um cliente. Isso ajuda os desenvolvedores a entenderem o formato da resposta.


Agora temos um método POST completo que permite cadastrar clientes e retorna uma confirmação após a criação.
- Request Body (CriaCliente): O corpo da requisição, contendo os dados obrigatórios (nome, idade, CPF).
- Response Body (ClienteCriado): O corpo da resposta, confirmando o sucesso da criação e retornando o ID do cliente criado.
- Exemplos: Fornecemos exemplos de como os dados devem ser enviados e como eles serão retornados, facilitando o entendimento e o desenvolvimento.
