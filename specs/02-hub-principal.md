# Spec: hub-principal

## Objetivo
Serviço orquestrador central. Executa a cada minuto, lê agendamentos pendentes no MongoDB central e publica eventos no RabbitMQ para as integrações corretas.

## Path
`hub-principal/`

## Módulo
```
github.com/user/my-hub-monorepo/hub-principal
```

## Comportamento
1. Conectar ao RabbitMQ usando `RABBITMQ_URL`.
2. Conectar ao MongoDB central usando `CENTRAL_MONGO_URL`.
3. Obter a collection `hub_central_scheduler.schedules`.
4. Iniciar `time.Ticker` de 1 minuto.
5. Na inicialização e a cada tick, executar `runCycle`:
   - Query: `{ next_run: { $lte: now } }`.
   - Para cada documento, montar `EventPayload`.
   - Resolver a fila via `resolveQueue`.
   - Chamar `PublishEvent` na fila correspondente.
6. Logar SUCESSO/INFO/ERRO via shared-core.

## Funções Principais
```go
func runCycle(ch *amqp.Channel, coll *mongo.Collection)
func resolveQueue(integration string) string
func getEnv(key, defaultValue string) string
```

## Mapeamento de Filas
| Integration | Fila |
|---|---|
| `integracao-a`, `A` | `fila_integracao_a` |
| `integracao-b`, `B` | `fila_integracao_b` |
| outro | "" (erro logado) |

## Variáveis de Ambiente
- `RABBITMQ_URL`
- `CENTRAL_MONGO_URL`

## Dockerfile
Multi-stage build Alpine com CA certificates. Copia `shared-core` e `hub-principal`, executa `go mod download` e `go build`.

## Critérios de Aceitação
- [ ] Publica eventos apenas para agendamentos com `next_run <= now`.
- [ ] Ignora integrações desconhecidas sem interromper o ciclo.
- [ ] Não dá panic em erros transitórios de RabbitMQ/MongoDB; loga e continua.
- [ ] Container inicia e mantém o cron rodando.
