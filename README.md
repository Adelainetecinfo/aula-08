# Sistema de Gerenciamento Acadêmico (SGA)

Projeto Integrador da disciplina **Programação Web II**: API REST em Java com Spring Boot para gerenciamento de alunos, cursos e matrículas.

Esta entrega corresponde à **Aula 07: Camada de Serviço, Regras de Negócio e Transações**, desenvolvida de forma incremental sobre o código das aulas anteriores.

- **Curso:** Ciência da Computação / Tecnólogo em Análise e Desenvolvimento de Sistemas
- **Docente:** Josemar Cristiano da Silva
- **Autores:**
  1. Paulo Zanardi
  2. Nawan Anderson
  3. Adelaine Rodrigues

---

## Tecnologias

- Java 17+ (ajuste para a versão usada no projeto)
- Spring Boot
- Spring Web
- Spring Data JPA / Hibernate
- Banco de dados: _H2 / MySQL / PostgreSQL (ajuste conforme o projeto)_
- JUnit 5 e Mockito
- Maven

---

## Arquitetura

O projeto segue a separação em camadas:

```
Controller  →  Service  →  Repository  →  Banco de Dados
```

| Camada | Responsabilidade |
|---|---|
| **Controller** | Recebe as requisições HTTP e devolve as respostas. Não contém regra de negócio. |
| **Service** | Aplica as regras de negócio, lança exceções e controla as transações (`@Transactional`). |
| **Repository** | Executa as operações de persistência com Spring Data JPA. |
| **Banco de Dados** | Armazena os registros acadêmicos. |

---

## O que foi implementado na Aula 07

### Matrícula (`MatriculaService`)

- Verifica se o aluno e o curso existem.
- Impede matrícula **ativa duplicada** do mesmo aluno no mesmo curso.
- Registra a matrícula com status `ATIVA`.

### Exceções de negócio (`RegraNegocioException`)

Exceção não verificada (`RuntimeException`) com mensagens específicas para:

- aluno não encontrado;
- curso não encontrado;
- matrícula ativa já existente;
- operação incompatível com o status atual da matrícula.

### Estados da matrícula

| Operação | Estado inicial | Estado final |
| Trancar  |     `ATIVA`    |   `TRANCADA` |
| Cancelar |     `ATIVA`    |  `CANCELADA` |
| Concluir |     `ATIVA`    |  `CONCLUIDA` |

Qualquer transição diferente das previstas lança `RegraNegocioException`.

### Transações e dirty checking

- Os métodos de serviço que alteram dados usam `@Transactional`.
- Em caso de exceção não verificada, o Spring faz **rollback** e as alterações anteriores da transação são desfeitas.
- A alteração de status usa uma entidade gerenciada pelo JPA: ao modificar o atributo `status` dentro da transação, o Hibernate detecta a mudança (**dirty checking**) e executa o `UPDATE` no commit, sem chamada explícita a `save()`.

### Exclusão de aluno e curso

- Não é possível excluir aluno com matrícula **ativa**.
- Não é possível excluir curso com matrícula **ativa**.
- Quando as regras e as restrições de integridade permitem, a exclusão é realizada.
- Quando não é permitida, retorna-se uma mensagem de erro apropriada.

### Tratamento de exceções na API

A `RegraNegocioException` é convertida em resposta HTTP com mensagem clara (por exemplo, `400 Bad Request` ou `404 Not Found`) por um `@RestControllerAdvice`.

---
  
### 1. Clonar o repositório

```bash
git clone https://github.com/SEU-USUARIO/NOME-DO-REPOSITORIO.git
cd NOME-DO-REPOSITORIO
```

### 2. Configurar o banco de dados

Edite `src/main/resources/application.properties` conforme o seu ambiente.

Exemplo com H2 em memória:

```properties
spring.datasource.url=jdbc:h2:mem:sga
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=update
spring.h2.console.enabled=true
```

Exemplo com MySQL:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/sga
spring.datasource.username=SEU_USUARIO
spring.datasource.password=SUA_SENHA
spring.jpa.hibernate.ddl-auto=update
```

### 3. Executar a aplicação

```bash
./mvnw spring-boot:run
```

ou, com o Maven instalado:

```bash
mvn spring-boot:run
```

A API ficará disponível em `http://localhost:8080`.

### 4. Executar os testes

```bash
./mvnw test
```

---

## Testes unitários

Os testes do `MatriculaService` usam **JUnit 5** e **Mockito** (repositórios simulados com mocks) e cobrem:

| Cenário | Resultado esperado |
|---|---|
| Aluno e curso existentes | Matrícula criada |
| Aluno inexistente | Exceção |
| Curso inexistente | Exceção |
| Matrícula ativa duplicada | Operação rejeitada |
| Trancamento válido | Status `TRANCADA` |
| Conclusão inválida | Exceção |
| Cancelamento válido | Status `CANCELADA` |

---

## Endpoints principais

> Ajuste os caminhos conforme os controllers do seu projeto.

| Método | Rota | Descrição |
|---|---|---|
| `POST` | `/matriculas` | Cria uma matrícula (`alunoId`, `cursoId`) |
| `PATCH` | `/matriculas/{id}/trancar` | Tranca a matrícula |
| `PATCH` | `/matriculas/{id}/cancelar` | Cancela a matrícula |
| `PATCH` | `/matriculas/{id}/concluir` | Conclui a matrícula |
| `DELETE` | `/alunos/{id}` | Exclui um aluno (se permitido) |
| `DELETE` | `/cursos/{id}` | Exclui um curso (se permitido) |

### Exemplo de requisição

```bash
curl -X POST http://localhost:8080/matriculas \
  -H "Content-Type: application/json" \
  -d '{"alunoId": 1, "cursoId": 1}'
```

### Exemplo de resposta de erro

```json
{
  "mensagem": "Aluno já possui matrícula ativa neste curso."
}
```

---

## Estrutura do projeto

```
src/
├── main/java/.../
│   ├── controller/     # Endpoints REST
│   ├── service/        # Regras de negócio e transações
│   ├── repository/     # Persistência (Spring Data JPA)
│   ├── model/          # Entidades e enum StatusMatricula
│   └── exception/      # RegraNegocioException e handler global
└── test/java/.../
    └── service/        # Testes unitários (JUnit 5 + Mockito)
```

---

## Observações

- O código foi desenvolvido sobre o fonte da Aula 07, preservando as funcionalidades das aulas anteriores.
- O projeto continuará evoluindo como Projeto Integrador da disciplina.
