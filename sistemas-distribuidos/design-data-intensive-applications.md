# Design Data Intensive Applications

## Formats for Enconding Data

1.  Uma aplicação em execução mantém os dados em memória e em formato de Objectos, Listas, Arrays, HashTables, Árvores, etc. Essas estruturas são utilizadas pois permitem que a CPU às acessem e manipule de maneira eficiente;
2.  Quando queremos trafegar esses dados de um local a outro via rede, é necessário algum processo para agrupar as estruturas de dados em um sequência de _bytes -_ afinal, não faz sentido enviar um ponteiro de memória para outra máquina 😅.

> Esse processo de transformar as estruturas de dados em bytes é conhecida como _Encoding_ (também pode ser chamada de _Serialization_ ou _Marshalling_). O processo inverso, transformar _bytes_ em estrutura de dados, é conhecido como _Decoding_ (também pode ser chamado de _Deserialization_ ou _Unmarshalling_).

### Encoding padrão das linguagens

Por padrão as linguagens suportam converter objetos em _bytes_. Por exemplo: Java tem _java.io.Serializable_, Python tem _pickle e_ Ruby tem _Marshal._

Esse suporte padrão é uma maneira de salvar e depois restaurar um objeto em memória com pouco código adicional. Todavia, temos alguns problemas com essa abordagem:

-   São acopladas a linguagem; O binário resultante do _encoding_ do _pickle_ (Python) não é entendido no Java;
-   Oferece um risco de segurança; Uma vez que algum _hacker_ entende o binário, é possível manipula-lo para que instancie alguma classe maliciosa;
-   São ineficientes; Consomem muita CPU para fazer _encoding_ e _decoding;_

### JSON, XML e variações binárias

Deixando de lado os encoders padrões, temos os grandes rivais XML e JSON. XML é muito criticado por sua verbosidade. JSON é mais popular por ser suportado nos _browsers_ e nativo no JavaScript.

-   XML, JSON e CSV são formatos textuais em um formato legível.
-   Em CSV e XML, não é possível distinguir números de strings; JSON difere números de strings, mas não diferencia integers de floats, também não especifica precisão dos números;
-   JSON e XML suportam caracteres _unicode_, mas não suporta texto em binário (texto não _encoded_ em um caracter). Binário em string é útil, o _workaround_ é feito convertendo o binário em string usando Base64 - isso funciona mas aumenta em 33% o tamanho do payload;

**Formatos binários**

Para uso interno na organização é uma excelente alternativa pois é mais compacto e mais rápido de parsear.

> Existe maneiras de transferir o JSON ou XML em formato binário. BSON, BJSON, BISON e WBXML são famosos mas não são tão utilizados.

## Thrift e Protocol Buffers

Thrift e Protocol Buffers são formatos binários. Thrift criado pelo Facebook e Protocol Buffers pelo Google.

```yaml
# Thrift
struct Artist {
  1: required string       name,
  2: optional i64          user_id,
  3: optional list<string> musicianStyles
}

# Protocol Buffers
message Artist {
  required string name            = 1;
  optional int64  user_id         = 2;
  repeated string musician_styles = 3;
}
```

**Thrift CompactProtocol**

_sequencia de bytes (34 bytes)_

18 | 06 | 4d 61 72 74 69 6e | 16 | f2 14 | 19 | 28 | 0b | 64 61 79 64 72 65 61 6d 69 6e 67 | 07 | 68 61 63 6b 69 6e 67 | 00

significa:

## Avro

Apache Avro é um outro formato binário que difere do Thrift e do Protocol Buffers. Surgiu por volta de 2009 no projeto do Hadoop onde o Thrift não dava conta do recado.

-   Diferente dos demais formatos binários, o Avro não utiliza tags para identificar os campos;
-   A mesma mensagem encodada com o Avro tem apenas 32 bytes de comprimento, a mais compacta entre os formatos binários;
-   É o mais compacto por não transferir informação de campo na mensagem. A mensagem é simplesmente uma concatenação de valores;
-   Para identificar qual o campo e seu tipo é utilizado o _schema_. Isso significar que se a mensagem não seguir exatamente o _schema_ de quem está lendo, não será possível decodar com sucesso;
-   A compatibilidade entre as versões do _schema_ é feita com base nos _default values_ ou _null_ _values_.
    -   Adicionar um campo que não tem valor padrão, significa que novos leitores não conseguem ler mensagens de antigos escritores, logo são incompatíveis.
    -   Remover um campo que não tem valor padrão, significa que velhos leitores não conseguiram ler de novos escritores, logo se tornam incompatíveis.
-   _Null values_ são explicitamente declarados no _schema_, diferente dos demais protocolos que usam _optional_ e _required._
-   A negociação de versão de _schema_ pode ser feita de três maneiras diferentes:
    -   Incluir o _schema_ no começo da mensagem dentro de um _Container -_ hadoop;
    -   Incluir apenas o número (ou hash) da versão do _schema_ no começo da mensagem - database;
    -   Negociar a versão durante a abertura da conexão via rede (gRPC)

```avro
record Artist {
  string               name;
  union { null, long } userId;
  array<string>        musicianStyles;
}
```