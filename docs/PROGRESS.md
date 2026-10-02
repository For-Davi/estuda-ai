# Progresso

## Etapa atual
Fase 0 — Etapa 0.5: ArchUnit protegendo as camadas

## Concluídas
- [x] 0.1 Monorepo e Git — 2026-10-01
  - Aprendi: monorepo vs polyrepo, Conventional Commits, .gitignore, .gitkeep
  - Dúvidas: -
- [x] 0.2 Primeiro ADR — 2026-10-01
  - Aprendi: formato ADR, monólito modular, por que a IA fica separada
  - Dúvidas: -
- [x] 0.3 Docker Compose da infraestrutura — 2026-10-01
  - Aprendi: imagem vs container, volumes (down vs down -v), portas HOST:CONTAINER, .env fora do Git, healthcheck
  - Dúvidas: -
  - Decisão: MinIO trocado por RustFS (imagem do MinIO saiu do Docker Hub); variáveis com nome genérico S3_* — candidato a ADR 0002
- [x] 0.4 Esqueleto do backend Spring Boot — 2026-10-01
  - Aprendi: módulos × camadas (domain/application/infrastructure/presentation), inversão de dependência, Maven Wrapper, application.yml com profiles, @RestController e records como DTO
  - Dúvidas: -
  - Decisão: Spring Boot 4.1.1 (a linha 3.x saiu do suporte gratuito em 06/2026); o backend lê as senhas do mesmo infra/.env do Compose
