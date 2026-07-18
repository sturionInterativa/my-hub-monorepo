# Spec: shared-core

## Objetivo
Pacote Go reutilizável que centraliza models, helpers de log, conexão com MongoDB e operações RabbitMQ usados por `hub-principal` e pelas integrações.

## Path
`shared-core/`

## Módulo
```
github.com/user/my-hub-monorepo/shared-core
```

## Responsabilidades
1. Definir structs compartilhadas (`EventPayload`, `CentralSchedule`).
2. Fornecer funções de log padronizadas (`LogSucesso`, `LogInfo`, `LogErro`).
3. Abstrair conexão com MongoDB.
4. Abstrair conexão, publicação e consumo de mensagens RabbitMQ.

## API Pública

### Models
```go
type EventPayload struct {
    MessageID   string `json:"message_id" bson:"message_id"`
    TenantDB    string `json:"tenant_db" bson:"tenant_db"`
    Integration string `json:"integration" bson:"integration"`
}

type CentralSchedule struct {
    ID          string    `json:"id" bson:"_id"`
    MessageID   string    `json:"message_id" bson:"message_id"`
    TenantDB    string    `json:"tenant_db" bson:"tenant_db"`
    Integration string    `json:"integration" bson:"integration"`
    NextRun     time.Time `json:"next_run" bson:"next_run"`
}
```

### RabbitMQ
```go
const QueueIntegracaoA = "fila_integracao_a"
const QueueIntegracaoB = "fila_integracao_b"

func ConnectRabbitMQ(amqpURL string) (*amqp.Connection, *amqp.Channel, error)
func PublishEvent(ch *amqp.Channel, queue string, payload EventPayload) error
func ConsumeEvents(ch *amqp.Channel, queue string) (<-chan amqp.Delivery, error)
```

### MongoDB
```go
func ConnectMongoDB(uri string) (*mongo.Client, error)
```

### Helpers
```go
func LogSucesso(componente, msg string)
func LogInfo(componente, msg string)
func LogErro(componente, msg string)
```

## Dependências
```go
require (
    github.com/rabbitmq/amqp091-go v1.10.0
    go.mongodb.org/mongo-driver v1.17.3
)
```

## Critérios de Aceitação
- [ ] `go test ./...` passa sem erros de compilação.
- [ ] `go vet ./...` não reporta problemas.
- [ ] Não possui `main` nem código de negócio específico de integração.
- [ ] Todas as funções retornam `error` e não dão panic sozinhas.
