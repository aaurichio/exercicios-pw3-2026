# 🎟️ Atividade Prática: Desenvolvimento Incremental da API de Salas no Spring Boot
## Roteiro de Implementação por Etapas (Fatias Verticais) e Controle de Versão

**Disciplina:** Programação Web III (PW3)  
**Curso:** Técnico em Desenvolvimento de Sistemas  
**Instituição:** Etec Horácio Augusto da Silveira  
**Repositório Base:** `https://github.com/etechas/pw3-ingressos-av3`  
**Tecnologias:** Java 25, Spring Boot 4, Spring Data JPA, MapStruct, Lombok, Git/GitHub, H2 Database

---

## 🎯 Objetivo da Atividade

Desenvolver a API de gerenciamento de **Salas de Cinema** adotando o fluxo profissional de trabalho com **Git/GitHub** e o conceito de **fatias verticais** (*Vertical Slices*).

Em vez de construir todas as camadas de uma só vez, você implementará **uma funcionalidade completa de ponta a ponta por vez** (do Repositório até o Controller), testará no Postman/Insomnia e **realizará um commit específico para cada etapa**.

O fluxo de trabalho será estruturado da seguinte forma:
- 🚀 **ETAPA 0 (Pré-requisito):** Fork, Clone, Criação da Branch e Preparação Git
- 🟢 **ETAPA 1:** Listagem de todas as salas ativas (`GET /salas`) + **Commit 1**
- 🟡 **ETAPA 2:** Consulta de sala ativa por ID (`GET /salas/{id}`) + **Commit 2**
- 🔵 **ETAPA 3:** Cadastro de nova sala (`POST /salas`) + **Commit 3**

Para isso, você construirá as seguintes peças da arquitetura:
- **DTOs**: com `record` java
- **Mapper**: com MapStruct
- **Repository**: Spring Data JPA
- **Service**: regras e orquestração
- **Controller**: exposição da API HTTP

---

## 📐 Fluxo Arquitetural da Aplicação

Antes de iniciar o código, visualize o caminho percorrido pela informação em cada requisição:

```text
 [ Cliente HTTP: Postman / Front-end ]
              │   ▲
 (HTTP POST/GET)  │   │ (JSON / HTTP Status 200, 201, 404)
              ▼   │
     ┌───────────────────┐
     │  SalaController   │  <-- Camada Web / REST
     └───────────────────┘
              │   ▲
  (Passa DTO) │   │ (Retorna DTO)
              ▼   │
     ┌───────────────────┐
     │    SalaService    │  <-- Camada de Negócio
     └───────────────────┘
         │ ▲         │ ▲
(Usa DTO)│ │(Entid.) │ │(Salva / Busca Entidades)
         ▼ │         ▼ │
   ┌────────────┐  ┌────────────────┐
   │ SalaMapper │  │ SalaRepository │  <-- Camada de Dados
   └────────────┘  └────────────────┘
                           │ ▲
                           ▼ │ SQL
                   ┌───────────────────┐
                   │  Banco SQLServer  │
                   └───────────────────┘
```

---

## 🚀 ETAPA 0 (Pré-requisito): Preparação do Ambiente e Git

Antes de iniciar qualquer linha de código em Java, execute os passos de versionamento obrigatórios:

### 0.1 Fazer Fork do Repositório Oficial
1. Acesse o repositório base no GitHub: **[https://github.com/etechas/pw3-ingressos-av3](https://github.com/etechas/pw3-ingressos-av3)**
2. No canto superior direito da página, clique no botão **Fork**.
3. Escolha a sua conta pessoal (ou da dupla) no GitHub e confirme a criação do Fork.

### 0.2 Clonar o Repositório Localmente
No terminal da sua máquina, faça o clone do **seu** repositório forked (substitua `<seu-usuario>` pelo seu login do GitHub):
```bash
git clone https://github.com/<seu-usuario>/pw3-ingressos-av3.git
cd pw3-ingressos-av3
```

### 0.3 Criar e Trocar para a Branch de Avaliação
Crie uma nova branch seguindo estritamente o padrão de nomenclatura com a identificação dos integrantes da dupla:
```bash
git checkout -b alu1-alu2-av
```
*(Substitua `alu1` e `alu2` pelos nomes/primeiros nomes dos integrantes. Exemplo: `lucas-mariana-av`).*

Verifique se você está na branch correta:
```bash
git branch
# Deverá listar a branch com asterisco: * alu1-alu2-av
```

Altere o arquivo `README.md` com o nome das duplas.

> [!IMPORTANT]
> **Regra Obrigatória de Avaliação:**  
> A cada etapa concluída (código implementado e testado no Postman), você **deve** realizar o commit com a mensagem indicada no roteiro. O histórico de commits na branch será avaliado!

---

## 🗂️ Recursos Pré-Existentes no Projeto

Para esta atividade, considere que a base do banco de dados e o DTO de resposta **já foram criados previamente**:

### 1. Entidade JPA: `Sala.java`
Localizada em `br.com.etechoracio.ingresso.entity`:
```java
package br.com.etechoracio.ingresso.entity;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

import java.time.LocalDateTime;

@Getter
@Setter
@Entity
@Table(name = "TBL_SALA")
public class Sala {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "ID_SALA")
    private Long id;

    @Column(name = "TX_NOME")
    private String nome;

    @Column(name = "NR_PRECO")
    private Double preco;

    @Column(name = "DT_EXCLUSAO")
    private LocalDateTime dataExclusao;

}
```

> [!IMPORTANT]
> **Conceito de Soft Delete (Exclusão Lógica):**  
> No sistema `ingresso`, as salas não são apagadas fisicamente do banco de dados com `DELETE`. Quando uma sala é excluída, a coluna `DT_EXCLUSAO` é preenchida com a data/hora da exclusão. Portanto, **uma sala só é considerada ativa quando `dataExclusao` for NULA (`dataExclusao IS NULL`)**!

---

### 2. DTO de Resposta: `SalaResponseDTO.java`
Localizado em `br.com.etechoracio.ingresso.dto`:
```java
package br.com.etechoracio.ingresso.dto;

public record SalaResponseDTO(
        Long id,
        String nome,
        Double preco
) {
}
```
*(Este record já está pronto e será reutilizado como retorno em todas as etapas).*

---

## 🟢 ETAPA 1: Listagem de Todas as Salas Ativas (`GET /salas`)

Nesta etapa, você criará a estrutura mínima inicial para listar as salas ativas do cinema.

```text
Postman (GET /salas) ➔ SalaController ➔ SalaService ➔ SalaRepository ➔ Banco SQLServer
                            │              │              │
                       (Retorna        (Chama       (Executa SQL
                       List<DTO>)       Mapper)      WHERE DT_EXCLUSAO IS NULL)
```

### 1.1 Camada Repository
**Pacote:** `br.com.etechoracio.ingresso.repository`

1. Crie a interface `Repository` estendendo `JpaRepository`.
2. Adicione o *Query Method* que busca todas as salas ativas

### 1.2 Camada Mapper
**Pacote:** `br.com.etechoracio.ingresso.mapper`

1. Crie a interface `Mapper` com a anotação `@Mapper(componentModel = "spring")`.
2. Adicione o método para converter uma lista de entidades `Sala` para uma lista de `SalaResponseDTO`

### 1.3 Camada Service
**Pacote:** `br.com.etechoracio.ingresso.service`

1. Crie a classe `Service` anotada com `@Service`.
2. Implemente o método para buscar todas as salas ativas:
    - Chame o método criado no item 1.1 do `repository`.
    - Utilize o `Mapper` para converter a lista de entidades para DTOs.
    - Retorne a lista de `SalaResponseDTO`.

### 1.4 Camada Controller
**Pacote:** `br.com.etechoracio.ingresso.controller`

1. Crie a classe `Controller` com as anotações:
    - `@RestController`
    - `@RequestMapping`
    - `@CrossOrigin("*")`
2. Implemente o endpoint de listagem:
    - Anotação: `@GetMapping`
    - Retorno: `List<SalaResponseDTO>`
    - Ação: Invocar e retornar o resultado do método que busca todas as salas ativas do `Service`.

### 🧪 Teste Prático da Etapa 1
Inicie o projeto e realize a requisição:
- **Método HTTP:** `GET`
- **URL:** `http://localhost:8080/salas`
- **Status esperado:** `200 OK`
- **Corpo esperado:** Array JSON com as salas ativas cadastradas ou array vazio `[]`.

---

### 📌 Commit da Etapa 1
Após validar o funcionamento da listagem, registre seu progresso no Git:
```bash
git add .
git commit -m "feat: etapa 1 - listagem de todas as salas ativas (GET /salas)"
git push -u origin alu1-alu2-av
```

---

## 🟡 ETAPA 2: Consulta de Sala Ativa por ID (`GET /salas/{id}`)

Agora que a listagem funciona, implementaremos a busca pontual de uma sala pelo seu ID.

```text
Postman (GET /salas/1) ➔ SalaController ➔ SalaService ➔ SalaRepository ➔ Banco SQLServer
                              │               │              │
                         (Se achar: 200    (Retorna      (Busca por ID e
                          Se não: 404)     Optional)     DT_EXCLUSAO IS NULL)
```

### 2.1 Camada Repository
Na interface `Repository` de sala, adicione o método derivado que busca por ID garantindo que a sala não esteja excluída logicamente

### 2.2 Camada Mapper
Na interface `Mapper` de sala, adicione o método para converter **uma única entidade** `Sala` em `SalaResponseDTO`

### 2.3 Camada Service
Na classe `Service` de sala, implemente o método de busca por ID:
- **Lógica:** Chame o método criado no item 2.1 de `Repository` e faça o mapeamento para o `Optional` usando o `Mapper` para o `SalaResponseDTO`.

### 2.4 Camada Controller
Na classe `Controller` de sala, adicione o endpoint de busca por ID:
- **Anotação:** `@GetMapping("/{id}")`
- **Lógica:**
    - Invoque o método criado no item 2.3 de `Service`.
    - Se a sala for encontrada, retorne status **200 OK** com o DTO.
    - Se a sala não existir ou estiver excluída, retorne status **404 Not Found**.

### 🧪 Teste Prático da Etapa 2
1. **Cenário de Sucesso (ID existente):**
    - **Método:** `GET`
    - **URL:** `http://localhost:8080/salas/1`
    - **Status esperado:** `200 OK`
    - **Corpo:** Dados da sala em JSON.
2. **Cenário de Recurso Inexistente:**
    - **Método:** `GET`
    - **URL:** `http://localhost:8080/salas/999`
    - **Status esperado:** `404 Not Found` (sem corpo).

---

### 📌 Commit da Etapa 2
Após validar os dois cenários (200 e 404), realize o commit da etapa:
```bash
git add .
git commit -m "feat: etapa 2 - consulta de sala ativa por id (GET /salas/{id})"
git push origin alu1-alu2-av
```

---

## 🔵 ETAPA 3: Inserção de Dados / Cadastro de Nova Sala (`POST /salas`)

Nesta etapa, implementaremos o cadastro de novas salas no cinema. Aqui precisaremos criar o DTO de entrada.

```text
Postman (POST /salas) ➔ SalaController ➔ SalaService ➔ SalaRepository ➔ Banco SQLServer
     (com JSON no            │               │              │
      Request Body)     (Retorna 201    (Converte DTO   (save() gera ID
                          CREATED)      em Entidade)    no banco)
```

### 3.1 Camada DTO: Criar `Request DTO`
**Pacote:** `br.com.etechoracio.ingresso.dto`

Crie o Record `Request DTO` de sala com as informações enviadas pelo cliente:
- `nome`
- `preco`

> [!NOTE]
> Observe que o campo `id` **não** existe no `Request DTO`, pois ele é gerado automaticamente pelo banco de dados (`IDENTITY`), e a `dataExclusao` também não existe, pois novas salas nascem ativas (com data nula).

### 3.2 Camada Mapper
Na interface `Mapper`, adicione o método para converter o `Request DTO` criado para a entidade `Sala`

### 3.3 Camada Service
Na classe `Service`, implemente o método de criação:
- **Lógica:**
    1. Converta o `Request DTO` recebido para entidade com `Mapper`.
    2. Salve a entidade no banco com `save de Repository`.
    3. Converta a entidade salva (que agora possui o ID gerado) para `SalaResponseDTO` com `Mapper`.
    4. Retorne o DTO de resposta.

### 3.4 Camada Controller
Na classe `Controller`, adicione o endpoint de criação:
- **Anotação:** `@PostMapping`
- **Lógica:**
    - Chame o método criado no item 3.3 em `Service`.
    - Retorne a resposta com o código de status HTTP **201 Created**

### 🧪 Teste Prático da Etapa 3
No Postman / Insomnia:
- **Método HTTP:** `POST`
- **URL:** `http://localhost:8080/salas`
- **Headers:** `Content-Type: application/json`
- **Corpo da Requisição (raw JSON):**
  ```json
  {
    "nome": "Sala 3 - 4DX VIP",
    "preco": 55.00
  }
  ```
- **Status HTTP esperado:** `201 Created`
- **Corpo da Resposta esperado:**
  ```json
  {
    "id": 1,
    "nome": "Sala 3 - 4DX VIP",
    "preco": 55.0
  }
  ```
- **Verificação final:** Execute novamente o `GET /salas` da Etapa 1 e confirme que a nova sala aparece na listagem!

---

### 📌 Commit da Etapa 3
Após validar o cadastro com sucesso, registre o commit final da funcionalidade:
```bash
git add .
git commit -m "feat: etapa 3 - cadastro de nova sala (POST /salas)"
git push origin alu1-alu2-av
```

---

## 📦 Entrega da Atividade

Para submeter a atividade:
1. Certifique-se de que todos os commits foram enviados para o seu repositório remoto no GitHub:
   ```bash
   git status
   git log --oneline -n 4
   ```
2. Compartilhe com o professor as alterações da sua branch no GitHub através da abertura de PR.

---

## 📊 Rubrica de Avaliação

| Critério / Etapa | Descrição e Requisitos                                                                                               | Pontuação |
| :--- |:---------------------------------------------------------------------------------------------------------------------| :---: |
| **Etapa 0: Git & GitHub** | Fork realizado com sucesso, branch nomeada no padrão `alu1-alu2-av` e commits individuais identificando cada etapa.  | **I** |
| **Etapa 1: Listagem (`GET /salas`)** | `Repository`, `Mapper`, `Service` e `Controller` retornando lista ativa com status 200 OK.                           | **R** |
| **Etapa 2: Consulta por ID (`GET /salas/{id}`)** | `Repository`, `Mapper`, `Service` e `Controller` tratando `Optional` com `ResponseEntity` (200 OK vs 404 Not Found). | **B** |
| **Etapa 3: Cadastro (`POST /salas`)** | Criação do `Request DTO`, métodos no Mapper e Service, e `Controller` com `@RequestBody` e status 201 Created.   | **MB** |

---
