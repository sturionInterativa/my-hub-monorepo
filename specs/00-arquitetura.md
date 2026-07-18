# Spec: Arquitetura SaaS Multi-tenant do HUB

## Contexto
O HUB migrou de um modelo monolítico por cliente para uma arquitetura SaaS multi-tenant no Kubernetes, orientada a eventos via RabbitMQ e com MongoDB usando estratégia database-per-tenant.

## Componentes
- **hub-principal**: orquestrador central. Roda cron a cada minuto, consulta agendamentos centrais e publica eventos no RabbitMQ.
- **integracao-a**: consumidor da fila A. Processa eventos, alterna para o banco do tenant, executa chamada externa e atualiza agendamentos.
- **integracao-b**: consumidor da fila B. Equivalente à integração A, mas consome fila diferente e pode executar outro tipo de API externa.
- **shared-core**: biblioteca comum com models, helpers de log, conexão MongoDB e conexão/publicação/consumo RabbitMQ.

## Fluxo de Dados
1. `hub-principal` faz query em `hub_central_scheduler.schedules` filtrando `next_run <= now`.
2. Para cada agendamento, publica `EventPayload` na fila correta (`fila_integracao_a` ou `fila_integracao_b`).
3. A integração correspondente consome o evento, conecta no banco do tenant (`tenant_db`), lê `messages`, chama API externa e atualiza `next_run` no tenant e no central.

## Restrições e Decisões
- Cada integração mantém uma conexão única com o cluster MongoDB e alterna database por evento.
- O HUB não acessa bancos de tenant; ele usa apenas o banco central leve.
- RabbitMQ é o único canal de comunicação entre HUB e integrações.
- Todos os serviços são Go 1.22, empacotados em containers Alpine.

## Variáveis de Ambiente Globais
- `RABBITMQ_URL`
- `CENTRAL_MONGO_URL`
- `TENANT_MONGO_URL`

## Critérios de Aceitação
- [ ] Clone com submodules traz todos os repositórios filhos.
- [ ] `go build` funciona em cada módulo dentro do workspace.
- [ ] Dockerfiles geram imagens enxutas com CA certificates.
