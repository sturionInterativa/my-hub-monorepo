# my-hub-monorepo

Monorepo principal que vincula os repositórios filhos via Git submodules:

- `shared-core`: funções e structs compartilhadas
- `integracao-a`: consumidor RabbitMQ da Integração A
- `integracao-b`: consumidor RabbitMQ da Integração B
- `hub-principal`: orquestrador central com cron e publicador de eventos

## Clonar

```bash
git clone --recurse-submodules https://github.com/sturionInterativa/my-hub-monorepo.git
```

## Estrutura

```
my-hub-monorepo/
├── go.work
├── .env.example
├── .gitmodules
├── shared-core/
├── integracao-a/
├── integracao-b/
└── hub-principal/
```
