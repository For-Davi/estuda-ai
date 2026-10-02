# Estuda.ai

> Plataforma de estudos para concursos públicos guiada pelo edital.

![status](https://img.shields.io/badge/status-em%20desenvolvimento-yellow)

Você envia o PDF do edital e a IA extrai cargos, vagas, salários, datas e o conteúdo programático. Depois você revisa os dados, escolhe o cargo e diz quantas horas tem por dia. A partir disso, o Estuda.ai monta um cronograma com revisões espaçadas até a data da prova, oferece resumos e questões por tópico e mantém você estudando com ofensiva e XP, no estilo do Duolingo.

## Como funciona

1. **Upload do edital:** o PDF é armazenado e processado de forma assíncrona.
2. **Extração com IA:** um LLM com saída estruturada (validada por schema) identifica cargos, datas e matérias.
3. **Revisão:** o usuário corrige o que a IA errou e confirma.
4. **Cronograma:** distribuição das horas por prioridade (peso × nº de questões), com revisões em +1, +7 e +30 dias.
5. **Estudo:** resumos e questões gerados por tópico, com registro de desempenho.
6. **Gamificação:** ofensiva diária calculada no fuso do usuário, XP e metas.

## Arquitetura

O backend é um monólito modular em Java. O serviço de IA fica separado, em Python, e os dois se comunicam de forma assíncrona via RabbitMQ.

```
React (PWA) ──REST/JWT──► Spring Boot (monólito modular) ──► PostgreSQL / MongoDB
                                  │                    ▲
                           publica job          publica resultado
                                  ▼                    │
                               RabbitMQ ◄──────► IA Python (FastAPI) ──► LLM / MinIO
```

- **Backend:** módulos `identity`, `edital`, `planning`, `study`, `gamification` e `billing`, cada um em camadas (`domain → application → infrastructure / presentation`). O `domain` é Java puro, e essa regra é verificada por testes com ArchUnit.
- **Serviço de IA:** fica separado por usar outra linguagem e ter outro perfil de carga (processamento longo, custo por token).

As decisões de arquitetura estão registradas em [`docs/adr/`](docs/adr/).

## Stack

| Parte | Tecnologias |
|---|---|
| Backend | Java 21, Spring Boot 3, Spring Security + JWT, JPA, MongoDB, Flyway, AMQP |
| Frontend | React, TypeScript, Vite, TanStack Query, Tailwind + shadcn/ui, React Hook Form + Zod |
| IA | Python 3.12, FastAPI, Pydantic, PyMuPDF, aio-pika, uv |
| Infra | Docker Compose, PostgreSQL, MongoDB, RabbitMQ, MinIO, GitHub Actions |
| Testes | JUnit 5, Mockito, AssertJ, Testcontainers, ArchUnit, Vitest, pytest |

## Estrutura do repositório

```
.
├── frontend/      # SPA React
├── backend/       # API Spring Boot (monólito modular)
├── ai-service/    # Serviço de IA em Python
├── infra/         # Docker Compose e configs de infraestrutura
├── docs/          # ADRs, diagramas e progresso
└── .github/       # Workflows de CI
```

## Como rodar

> 🚧 Em construção. As instruções entram aqui quando o `docker compose up` estiver subindo os serviços.

## Roadmap

- [ ] **Fase 0:** Fundação (infra, esqueletos, autenticação JWT, CI)
- [ ] **Fase 1:** Edital (upload, extração com IA, revisão)
- [ ] **Fase 2:** Cronograma (algoritmo, revisões espaçadas, calendário, `.ics`)
- [ ] **Fase 3:** Estudo (resumos, questões, desempenho)
- [ ] **Fase 4:** Gamificação (ofensiva, XP, metas)
- [ ] **Fase 5:** Produção (outbox, resiliência, observabilidade, deploy)

## Autor

Carlos Davi
