
# Introdução <a name="introduction"></a>

Elasticsearch é um banco de dados distribuído para realizar buscas e análise de dados em tempo real. É muito comum ser utilizado para *fulltext search*, *structured search*, *analytics*, *geoespaciais*, etc. Alguns dos players de mercado que fazem uso e você possa conhecer:
- Wikipedia
- Github
- StackOverflow
- The Guardians
- Datadog

## Orientado a documentos

Diferente dos bancos de dados relacionais que aprendemos na faculdade e são orientados a tabelas, colunas e linhas, o Elasticsearch é orientado a documento, o que significa que o dado é armazenado como um objeto inteiro em formato JSON. A escolha da notação JSON é simplesmente pelo fato de que é amplamente suportado pelas linguagens modernas e de ser usado como padrão no movimento NoSQL.

## Armazenamento de documentos

No Elasticsearch, um documento pertence a um *type* e um *type* pertence a um *index*. Chamamos o ato de armazenar um documento de *indexing*. Fazendo um paralelo aos bancos de dados relacionais, temos:

>   DB Relacionais  ⇒ Schema ⇒ Table   ⇒ Rows            ⇒ Columns
>   Elasticsearch     ⇒ Indices   ⇒ Types  ⇒ Documents ⇒ Fields

Um cluster de Elasticsearch pode conter múltiplos *indices (databases)*, que podem conter múltiplos *types (tables)*. Esses *types* podem conter mútiplus *documents (rows)* e cada *document* contém múltiplos *fields (columns)*.

# Executando com Docker <a name="running-docker"></a>

Para rodar o Elasticsearch local com Docker, estou utilizando o seguinte `docker-compose`:
```yml
version: '3'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.7.0
    container_name: elk_es01
    restart: on-failure
    volumes:
	  - ./es/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    environment:
      - "discovery.type=single-node"
    ports:
      - '9200:9200'
      - '9300:9300'
```

com o seguinte arquivo de configuração:
```yml
cluster.name: foo-cluster  
  
node.name: es01  
  
network:  
  host: 0.0.0.0
  
path:  
  data: /usr/share/elasticsearch/data  
  logs: /usr/share/elasticsearch/log

xpack.security.enabled: true # flag importante para evitar erro de /usr/share/elasticsearch/config/elasticsearch.yml: Device or resource busy
```

> Para usar normalmente no mabiente de desenvolvimento não é necessário passar um arquivo de configuração, mas como vamos explorar através dos aprendizados do livro, estou deixando esse arquivo com o básico da configuração.

Para verificar se o Elasticsearch está de pé e pronto para uso, basta executar o curl abaixo:
```sh
curl 'http://localhost:9200/?pretty'
```

Como resposta deve-se observar um JSON parecido com esse:
```json
{
    "name": "es01",
    "cluster_name": "foo-cluster",
    "cluster_uuid": "_AKO7wRGSTWuQHwIieffkQ",
    "version": {
        "number": "7.17.9",
        "...",
        "lucene_version": "8.11.1"
    },
    "tagline": "You Know, for Search"
}
```

### Acessando o container <a name="accessing-the-container"></a>

Para acessar o container do Elasticsearch e explorar o conteúdo, pode utilizar o comando:
```shell
docker exec -i -t elk_es01 bash
```

# Manipulando documentos via REST API

## Indexando dados

Para começar a entender melhor sobre o funcionamento da ferramenta, que tal brincarmos um pouco com ela? O exemplo inicial do livro utiliza a seguinte estrutura representando dados de um usuário de uma empresa:

```json
{
    "email":      "john@smith.com",
    "first_name": "John",
    "last_name":  "Smith",
    "info": {
        "bio":         "Eco-warrior and defender of the weak",
        "age":         25,
        "interests": [ "dolphins", "whales" ]
    },
    "join_date": "2014/05/01"
}
```

Para o exemplo do quadro de funcionários de uma empresa, vamos realizar as seguintes operações:
- Indexar um documento contendo os detalhes de um funcionário.
- Cada documento será do tipo *employee*.
- O tipo estará contido no índice *megacorp*.
- Esse índice ficará no nosso *foo-cluster*.

Parece complexo, mas é bastante simples. Basta executar o seguinte comando:
```shell
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

Repare que das operações mencionadas, nenhuma precisou ser feita manualmente. O Elasticsearch executa tarefas administrativas automaticamente após as operações. No caso, através do caminho `/{index}/{type}/{id}` o Elasticsearch criou todos os recursos para nós.

## Buscando dados

Uma vez persistido, podemos mudar o verbo para `GET` para obter o documento através de seu ID

```shell
GET /megacorp/employee/1
```

**resposta**
```JSON
{
    "...",
    "_source": {
        "first_name": "John",
        "last_name": "Smith",
        "age": 25,
        "about": "I love to go rock climbing",
        "interests": [
            "sports",
            "music"
        ]
    }
}
```

Reparem que não retornou apenas o documento que foi enviado, mas sim toda a estrutura de metadados que o Elasticsearch usa para indexação dos documentos, junto do atributo `_source` o qual esse sim é o recurso enviado via `PUT`. Da mesma forma como o `GET`, é possível utilizar `HEAD` e `DELETE` para verificar se existe ou deletar um documento.

### Busca simples

O forte do Elasticsearch são suas poderosas buscas. A busca simples é feita com *query-string params*, exemplo:
```shell
GET /megacorp/employee/_search?q=last_name:Smith
```

**response**
```json
{
	"...",
    "hits": {
        "...",
        "hits": [
            {
                "...",
                "_source": {
                    "first_name": "John",
                    "last_name": "Smith",
                    "age": 25,
                    "about": "I love to go rock climbing",
                    "interests": [
                        "sports",
                        "music"
                    ]
                }
            }
        ]
    }
}
```

### Busca com DSL

A busca simples é útil, mas bastante limitada. A mais útilizada sem dúvida alguma é a busca com a _domain-specific language_ (DSL) do Elasticsearch, que permite construir buscas mais robustas e complexas.

```shell
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

**response**
```json
{
	"...",
    "hits": {
        "...",
        "hits": [
            {
                "...",
                "_source": {
                    "first_name": "John",
                    "last_name": "Smith",
                    "age": 25,
                    "about": "I love to go rock climbing",
                    "interests": [
                        "sports",
                        "music"
                    ]
                }
            }
        ]
    }
}
```

#### Full-Text search

Claro que além de busca por termo específico, as buscas do tipo *full-text search* se destacam nas características principais do Elasticsearch, exemplo:

```shell
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "go climbing"
        }
    }
}
```

**response**
```json
{
	"...",
    "hits": {
        "...",
        "hits": [
            {
                "...",
                "_source": {
                    "first_name": "John",
                    "last_name": "Smith",
                    "age": 25,
                    "about": "I love to go rock climbing",
                    "interests": [
                        "sports",
                        "music"
                    ]
                }
            }
        ]
    }
}
```

#### Phrase search

Encontrar texto com base nas palavras é muito bom, mas as vezes precisamos buscar pela sequência exata de palavras e o Elasticsearch também suporta, exemplo:

```shell
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

**response**
```json
{
	"...",
    "hits": {
        "...",
        "hits": [
            {
                "...",
                "_source": {
                    "first_name": "John",
                    "last_name": "Smith",
                    "age": 25,
                    "about": "I love to go rock climbing",
                    "interests": [
                        "sports",
                        "music"
                    ]
                }
            }
        ]
    }
}
```

### Busca analítica

Além das buscas já apresentadas, o banco oferece uma busca analítica conhecida como *aggregations* o que te permite gerar visualições sofisticadas em cima dos dados armazenadas. É uma espécie de `GROUP BY` conhecido nos bancos relacionais.

```shell
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```
