##  Desafio Técnico Sênior: Plataforma de Microcrédito Instantâneo

Este desafio visa avaliar a capacidade do candidato de projetar, implementar e garantir a robustez de um sistema distribuído de alto desempenho e tolerância a falhas, utilizando toda a stack fornecida em um contexto financeiro rigoroso.

-----

###  O Cenário: Microcrédito Instantâneo "CréditoJá"

Sua tarefa é projetar e desenvolver um **Serviço de Processamento de Propostas de Microcrédito Instantâneo** para o "CréditoJá". O objetivo principal é receber propostas de crédito, realizar verificações de elegibilidade (simuladas), notificar o usuário e garantir que o processamento seja **atômico, rastreável e rápido**.

### Stack Mandatória

| Categoria | Tecnologia | Uso Esperado no Desafio |
| :--- | :--- | :--- |
| **Backend** | **Java 17, Spring Boot** | Estrutura da aplicação, APIs RESTful e lógica de negócio. |
| **Banco de Dados** | **Postgres, Liquibase** | Persistência de dados transacionais. **Liquibase** para versionamento de schema. |
| **Cache/Mensageria** | **Redis, Kafka** | **Redis** para caching e controle de rate-limit (opcional). **Kafka** para comunicação assíncrona/processamento de eventos. |
| **Infra/DevOps** | **Docker, Terraform** | **Docker** para containerização da aplicação e serviços (Postgres, Redis, Kafka). **Terraform** para provisionar a infraestrutura necessária (Rede, Banco de Dados, etc., de forma simulada ou em nuvem se o avaliador permitir). |
| **Testes** | **JUnit, Mockito** | Testes unitários de serviços e classes, e testes de integração com banco de dados simulados. |
| **Observabilidade** | **Datadog** | Instrumentação da aplicação com métricas, logs e traces distribuídos. |

-----

### Requisitos do Desafio

#### 1\. Microsserviço de Proposta (`proposta-service`)

  * **API REST (`POST /api/propostas`):** Recebe uma solicitação de microcrédito:
    ```json
    {
      "clienteId": "UUID do Cliente",
      "valorSolicitado": 500.00,
      "prazoEmMeses": 6
    }
    ```
  * **Processamento Inicial:**
    1.  Salva a proposta no **Postgres** com o status inicial de `PENDENTE`.
    2.  Gera um evento `PROPOSTA_RECEBIDA` e o envia para o **Kafka**.
    3.  A API deve retornar um `202 Accepted` com a localização da proposta (`/api/propostas/{id}`).

#### 2\. Microsserviço de Análise (`analise-service`)

  * **Consumidor Kafka:** Assina o tópico `PROPOSTA_RECEBIDA`.
  * **Simulação de Análise:** Ao receber o evento, simula uma verificação de elegibilidade:
      * **Regra de Nível Sênior:** Use o **Redis** para simular uma *Blacklist de Clientes*. Se o `clienteId` estiver no Redis, a proposta é `REPROVADA`. Caso contrário, é `APROVADA`.
  * **Atualização e Notificação:**
    1.  Atualiza o status da proposta no **Postgres** (aprovada ou reprovada).
    2.  Gera um novo evento (`PROPOSTA_APROVADA` ou `PROPOSTA_REPROVADA`) e o envia para um tópico **Kafka** de notificação.

#### 3\. Persistência e Versionamento

  * Utilize **Postgres** como a fonte primária de dados.
  * Configure o **Liquibase** para gerenciar o schema do banco de dados (tabela `proposta` com `id`, `clienteId`, `valor`, `prazo`, `status`, `dataCriacao`).

#### 4\. Observabilidade e Monitoramento (Datadog)

  * Instrumente a aplicação para que o **Datadog** (simulado ou real) possa:
      * Capturar **logs estruturados** (JSON).
      * Registrar **métricas personalizadas** (ex: `proposta.analise.sucesso`, `proposta.analise.falha`).
      * Gerar **traces distribuídos** que liguem a chamada da API do `proposta-service` com o processamento no `analise-service` via Kafka.

#### 5\. Infraestrutura como Código (Terraform & Docker)

  * Crie um `docker-compose.yml` que provisione: a aplicação Spring Boot, Postgres, Redis e Kafka/Zookeeper.
  * Esboce um template **Terraform** (`main.tf`) que, em um ambiente real, criaria o *Cluster Kafka*, a *Instância do Postgres* e o *Cluster K8s* para a aplicação (não é necessário rodar o `apply`, mas a estrutura deve ser válida).

#### 6\. Qualidade e Testes

  * Crie **Testes Unitários** usando **JUnit** e **Mockito** para a lógica de negócio principal do `analise-service` (verificação de blacklist no Redis).
  * Crie **Testes de Integração** para garantir que o `proposta-service` consiga salvar no **Postgres** e enviar a mensagem para o **Kafka** (com `@SpringBootTest`).

-----

###  Entregáveis Finais (Foco Sênior)

O candidato deve apresentar:

1.  **Código-fonte** completo e funcional, seguindo boas práticas (Clean Code, Design Patterns, SOLID).
2.  **`docker-compose.yml`** para inicialização local da stack.
3.  **Arquivos `.tf` (Terraform)** com a infraestrutura abstrata.
4.  **Relatório/README.md** explicando:
      * O design de eventos (Schema Kafka).
      * O tratamento de **idempotência** e **falhas** no `analise-service` (ex: como garantir que uma proposta não seja analisada duas vezes em caso de reprocessamento do Kafka).

-----

### Critérios de Avaliação

| Critério | Descrição |
| :--- | :--- |
| **Arquitetura Distribuída** | Design eficiente da comunicação assíncrona com Kafka. Tratamento de *Dead Letter Queues (DLQs)* e **idempotência**. |
| **Resiliência e Performance** | Uso correto do **Redis** para cache/rate-limit. Configurações otimizadas do Spring Boot para alto *throughput*. |
| **Qualidade do Código** | Padrões de projeto, modularidade, clareza e manutenibilidade. Cobertura de testes significativa. |
| **Observabilidade** | Implementação avançada e correta de **Datadog** para rastrear a jornada completa da proposta. |
| **Infraestrutura como Código** | Design eficaz e realista dos templates **Terraform** e **Docker** para produção. |

-----

Este desafio é abrangente e foca na **integração de sistemas, resiliência e práticas de DevOps/Observabilidade**, que são essenciais para um desenvolvedor Java/Spring Sênior no setor financeiro.

Gostaria que eu detalhasse a especificação de algum dos tópicos, como o esquema de mensagens do Kafka ou a estrutura do Terraform?
