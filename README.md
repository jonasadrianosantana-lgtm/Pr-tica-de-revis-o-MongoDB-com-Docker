# Passo 18: Revisão das decisões (MongoDB, coleção `chamados`)

## Perguntas

**1. Qual informação pode ficar desatualizada por ter sido incorporada?**


Os dados de `solicitante`. Se a pessoa mudar de setor, chamados antigos mantêm o valor anterior. Alternativas: guardar só `solicitanteId` e usar `$lookup`, ou atualizar em massa com `updateMany`.

**2. O histórico poderia crescer sem limite? Como lidar?**
Sim, e o documento tem limite de 16 MB. Soluções:
- Mover o histórico para outra coleção (`chamados_historico`), um documento por evento.
- Manter só os últimos N eventos no documento e arquivar o resto.

**3. Qual consulta justificaria um índice em `marcadores`?**


Buscas frequentes por etiqueta, como `db.chamados.find({ marcadores: "hardware" })`, em coleções grandes. Cria-se com `createIndex({ marcadores: 1 })` (índice multikey).

**4. Quais campos devem ser obrigatórios em todas as categorias?**


`titulo`, `categoria`, `prioridade`, `status`, `solicitante` (`nome` e `setor`) e `criadoEm`. Use `$jsonSchema` com `enum` em `categoria`, `prioridade` e `status`.

**5. Quando o modelo documental deixaria de ser adequado?**


Quando o sistema exigir relacionamentos complexos entre muitas entidades, transações multi-entidade com forte consistência, dados sempre normalizados em um só lugar ou relatórios analíticos com muitos cruzamentos. Nesses casos, um banco relacional é mais seguro.

## Como Testar

**1. Confirme que Docker e Docker Compose estão disponíveis:**
```bash
docker --version
docker compose version
```

**1. Preparar o Docker Compose**
```yaml
services:
  mongo:
    image: mongo:8
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```
**3. Iniciar e observar o serviço**

```bash
docker --version
docker compose version
```
**4. Entrar no shell**
```bash
docker compose exec mongo mongosh -u admin -p admin --authenticationDatabase admin
```
No shell, confira os bancos:
```bash
show dbs
```
**5. Selecionar o banco**
```bash
use suporte
```
