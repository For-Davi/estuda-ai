# ADR 0001 — Monólito modular + serviço de IA separado

- **Status:** Aceito
- **Data:** 2026-10-01

## Contexto

- O Estuda.ai é construído por **uma pessoa**, com orçamento baixo e foco em chegar rápido a um MVP.
- O produto tem várias áreas de negócio: identidade, edital, cronograma, estudo, gamificação e cobrança. Elas mudam juntas com frequência no início.
- Uma parte do sistema é bem diferente das outras: o **processamento de editais com IA**.
  - O ecossistema de leitura de PDF e de SDKs de LLM é muito mais maduro em **Python** do que em Java.
  - O processamento leva de **30 s a 2 min** por edital, enquanto as demais requisições levam milissegundos.
  - Cada processamento **custa dinheiro** (tokens), e a demanda tem picos quando editais grandes são publicados.
- Operar muitos serviços (deploy, monitoramento, rede, consistência entre bancos) tem um custo alto para uma pessoa só.

## Decisão

Vamos construir o backend como um **monólito modular em Java (Spring Boot)** e o processamento de IA como um **serviço separado em Python (FastAPI)**, com comunicação **assíncrona via RabbitMQ**.

- O backend é dividido em módulos (`identity`, `edital`, `planning`, `study`, `gamification`, `billing`). Cada módulo tem as camadas `domain`, `application`, `infrastructure` e `presentation`.
- Um módulo não acessa a `infrastructure` de outro. Essa regra e a independência do `domain` em relação a frameworks são verificadas por testes com ArchUnit.
- A IA fica separada porque:
  1. usa outra linguagem, com bibliotecas melhores para o problema;
  2. tem outro perfil de carga (tarefas longas e caras);
  3. precisa escalar de forma independente (mais workers de IA sem duplicar a API).
- O backend publica um job na fila e responde na hora (`202 Accepted`). A IA consome o job e publica o resultado. A fila evita timeouts, absorve picos e deixa um lado funcionar mesmo com o outro fora do ar.

## Alternativas consideradas

### Microsserviços desde o início

Um serviço por módulo, cada um com o próprio banco e deploy.
**Descartada:** multiplica o custo de operação (pipelines, observabilidade, rede, transações distribuídas) sem trazer benefício para uma pessoa só. Além disso, as fronteiras entre os módulos ainda não estão estáveis, e errar uma fronteira entre serviços é muito mais caro do que errar uma fronteira entre pacotes.

### Monólito único (incluindo a IA em Java)

Tudo num só processo, com a extração feita em Java.
**Descartada:** as bibliotecas de PDF e LLM em Java são mais limitadas. Além disso, tarefas longas e pesadas disputariam recursos com a API, e escalar a IA obrigaria a escalar o sistema inteiro.

## Consequências

### Positivas
- Um único deploy do backend: simples de rodar, depurar e testar localmente.
- Transações locais entre módulos, sem consistência eventual onde ela não é necessária.
- Fronteiras claras entre módulos, protegidas por testes, que permitem extrair um módulo no futuro com pouco esforço.
- A IA escala, falha e é atualizada sem afetar a API.
- Cada parte usa a melhor ferramenta para o seu problema.

### Negativas
- Duas linguagens e dois ecossistemas para manter (build, testes, CI, dependências).
- A comunicação assíncrona traz complexidade: mensagens duplicadas, falhas na publicação (Outbox na Fase 5), contrato de mensagens entre Java e Python.
- O status do edital passa a ser eventual, e o front precisa acompanhar o processamento (polling ou SSE).
- O monólito exige disciplina: sem os testes de arquitetura, os módulos tendem a se acoplar com o tempo.

### Quando revisitar esta decisão
Um módulo deve virar um serviço separado quando aparecer pelo menos um destes sinais:
- precisa **escalar de forma diferente** do resto (ex.: gamificação recebendo muito mais eventos do que a API);
- precisa de um **ciclo de deploy próprio** e está atrasando os deploys dos outros módulos;
- passa a ser cuidado por **outro time**;
- tem **requisitos diferentes** de disponibilidade, segurança ou tecnologia.
