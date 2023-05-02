
# CIDR (Class Inter Domain Router), IPs Públicos e Privados

-   Método para alocação de IPv4s;
-   Ajuda a definir ranges de IPs;
    -   `/32` representa um único IP disponível.
    -   `/0` representa todos os IPs disponíveis.
    -   `192.168.0.0/26` representa um range de 64 IPs até o `192.168.0.63`

## Dois componentes principais

### Base do IP

É o integer que representa o range. Exemplo: `192.168.0.0` , `10.0.0.0`

### Máscara

Define quantos bits pode mudar no IP. Exemplo: `/0`, `/24`, `/32`

Pode ter duas formas:

-   `/8` que representa `255.0.0.0`
-   `/16` que representa `255.255.0.0`
-   `/24` que representa `255.255.255.0`
-   `/32` que representa `255.255.255.255`

### Resumo da tabela de IPs

-   `192.168.0.0` **/32** ⇒ Permite 1 IP (2^0) = `192.168.0.0`
-   `192.168.0.0` **/31** ⇒ Permite 2 IPs (2^1) = `192.168.0.0` → `192.168.0.1`
-   `192.168.0.0` **/30** ⇒ Permite 4 IPs (2^2) = `192.168.0.0` → `192.168.0.3`
-   `192.168.0.0` **/29** ⇒ Permite 8 IPs (2^3) = `192.168.0.0` → `192.168.0.7`
-   `192.168.0.0` **/28** ⇒ Permite 16 IPs (2^4) = `192.168.0.0` → `192.168.0.15`
-   `192.168.0.0` **/27** ⇒ Permite 32 IPs (2^5) = `192.168.0.0` → `192.168.0.31`
-   `192.168.0.0` **/26** ⇒ Permite 64 IPs (2^6) = `192.168.0.0` → `192.168.0.63`
-   `192.168.0.0` **/25** ⇒ Permite 128 IPs (2^7) = `192.168.0.0` → `192.168.0.127`
-   `192.168.0.0` **/24** ⇒ Permite 256 IPs (2^8) = `192.168.0.0` → `192.168.0.255`

...

-   `192.168.0.0` **/16** ⇒ Permite 65.536 IPs (2^16) = `192.168.0.0` → `192.168.255.255`

...

-   `192.168.0.0` **/0** ⇒ Permite Todos IPs = `0.0.0.0` → `255.255.255.255`

### Memorizar então:

IP é composto por 4 octetos: 1, 2, 3 e 4.

-   `/32` nenhum octeto pode mudar;
-   `/24` apenas o último octeto pode mudar;
-   `/16` apenas os dois últimos octetos podem mudar;
-   `/8` os três últimos octetos podem mudar;
-   `/0` todos os octetos podem mudar.

[](https://www.ipaddressguide.com/)[https://www.ipaddressguide.com/](https://www.ipaddressguide.com/)

## Public vs Private IP addresses

A IANA (Internet Assigned Numbers Authority) estipulou uma convenção de alguns ranges para IPs Privados:

-   `10.0.0.0/8` que vai do `10.0.0.0` ao `10.255.255.255` para grandes redes;
-   `172.16.0.0/12` que vai do `172.16.0.0` ao `172.31.255.255` → AWS default VPC;
-   `192.168.0.0/16` que vai do `192.168.0.0` ao `192.168.255.255` → Network de casa;
-   Todo o resto de IPs são públicos.

# VPC (Virtual Private Cloud)

## VPC padrão da AWS

-   Todas as contas possuem uma VPC padrão;
-   Todas as EC2 são lançadas nessa VPC caso não especificado algum subnet;
-   Essa VPC padrão tem acesso ao Internet Gateway e todas as EC2 são consequentemente públicas;

> A boa prática é criar uma VPC e não usar a padrão.

## Visão geral

-   Podemos ter até 5 VPCs por região - caso precise de mais, é possível solicitar;
-   Para cada VPC podemos ter até 5 CIDR, sendo:
    -   Mín de range é `/28`, o que permite 16 IPs;
    -   Máx de range é `/16`, o que permite 65.536 IPs;
-   Como a VPC é **sempre** privada, só podemos ter os três ranges de IP: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.

## Criando uma nova VPC

-   Ir até o menu de VPC e clicar em **Create VPC**
-   A convenção de nomes que utilizarei será `vpc-{region}-{environment}`: `vpc-us1-main`
-   CIDR que optei é o `10.0.0.0/16`, o que me da 65.536 IPs - que serão limitados nas subnets
-   Clicar em Criar.

# Subnets

-   Subnet é um sub-range de endereços de IP na VPC;
-   Em cada subnet, 5 IPs são reservados pela AWS e não pode ser utilizados (os quatro primeiros e o último):
    -   `10.0.0.0`: Network Address
    -   `10.0.0.1`: reservado pelo AWS VPC router
    -   `10.0.0.2`: reservado pelo mapeamento Amazon-provided DNS
    -   `10.0.0.3`: reservado para uso futuro da AWS
    -   `10.0.0.255`: Network Broadcast Address mas não é suportado nas VPCs e não é utilizado mas é reservado mesmo assim.
-   Ou seja, se escolhemos um CIDR de `/24` não teremos 256 IPs e sim 249 pois desconta os 5 acima.

## Criando as Subnets

Vamos criar duas subnets públicas e duas privadas.

### Subnet Públicas

-   Ir ate o menu Subnets e clicar em **Create Subnet**
-   Selecionar a VPC recém criada `vpc-us1-main`
-   A conversão de nome utilizei `{acesso}Subnet{AzRegion}`: `PublicSubnetA`
-   Escolher o Availability Zone: `us-east-1a`
-   O range CIDR utilizarei `10.0.0.0/24`, pois queremos limitar os IPs públicos.
    -   Lembrando que a máscara `/24` nos da 255 - 5 IPs o que é mais que suficiente para Web e LBs.
-   Repetir o mesmo processo par a a `PublicSubnetB`
-   O range CIDR utilizarei `10.0.1.0/24`, agora vai `1` no terceiro octeto para não conflitar com os 255 ranges da `PublicSubnetA`

### Subnet Privadas

-   Ainda na mesma tela, incluir uma nova subnet
-   O nome sera `PrivateSubnetA`
-   Escolher o Availability Zone: `us-east-1a`
-   O range CIDR utilizarei `10.0.16.0/20`, pois permite 4096 - 5 IPs que é muito mais que suficiente para uma pequena/média empresa e o terceiro octeto `16` para diferenciar das públicas.
-   Repetir o mesmo processo para a `PrivateSubnetB`
-   Com o Availability Zone: `us-east-1b`
-   E o range CIDR `10.0.32.0/20`, agora com 32 no terceiro octeto pois é a continuação da `PrivateSubnetA`

> No momento as subnets ainda são iguais, ainda não há distinção entre publica e privada na configuração.

> **Dica:** Aproveita que já está na tela de Subnets e já habilita o auto assign de IP público para as subnets públicas. `Clica na Subnet A → Modify auto-assign IP settings → Enable`

# IG - Internet Gateway

Internet Gateway é o que permite conectar os recursos (ex: EC2) à Internet.

-   Escala de maneira horizontal e é super High Available e Redundante, não precisamos nos preocupar.
-   É criado separadamente da VPC;
-   Um IG só pode ter uma VPC e vice-versa;
-   Por si só não permitem acesso a internet, necessita de configuração de Route Tables.

> No momento, se quiser testar a conectividade da sua VPC, pode fazer um teste e lançar um EC2 instancie e tentar conectar via ec2 Connect. Observará que não é possível ainda. **There was a problem connecting to your instancie**

## Criando um Internet Gateway

-   Navegar ate a aba Internet Gateway e clicar em **Create internet gateway;**
-   A convenção de nome que segui foi próxima ao da VPC `igw-{region}-{environment}`: `igw-us1-main`
-   O recém criado IG vem com status **Detached**, o qual vamos atrela-lo a nossa VPC `vpc-us1-main`
-   Selecione o IG, clique em **Attach to VPC,** selecione a VPC e confirma.

> Ainda não é possível conectar a nossa EC2, se quiser testar a conectividade fique a vontade mas seguirá observando que não é possível. Agora, precisamos do Route Table. **There was a problem connecting to your instancie**

## Criando um Route Table

-   Navegando até a página de Route Tables, perceberá que já existe uma para a VPC que criamos. Todavia, essa é a padrão, normalmente usada para subnets que não estão associadas a uma Route Table. Por hora, vamos deixar essa ai e criar duas novas mais explícitas: uma pública e uma privada.
-   Clicar em Create route table e preencher o nome. Segui a convenção anterior e a minha ficou: `rtb-us1-main-public`
-   Para a privada, repetir o procedimento mas agora o nome ficou `rtb-us1-main-private`

## Associando as Subnets as Route Tables

A route table pública, `rtb-us1-main-public`, terá associação das subnets públicas;

A route table privada, `rtb-us1-main-private`, terá associação das subnets privadas;

-   Para associar, basta clicar na Route Table
-   Clicar na **aba** Subnet associations e **Edit subnet associations**.

## Permitindo acesso da Internet a Route Table pública

-   Clicar na Route Table pública e clicar na aba **Route**.
-   Observará que já existe uma rota criada com destination `10.0.0.0/16` e target `local`
    -   Isso significa que todas as chamadas de um IP interno da VPC, será rotado internamente e não sairá para a Internet.
-   O que queremos fazer agora é adicionar uma rota nova, cuja destination é `0.0.0.0/0` - ou seja, qualquer IP público - e redirecionar para o Internet Gateway que criamos.

> Pronto! Agora você deveria ser capaz de acessar a instância EC2 através da Internet.

# Bastion TIME!

Você pode estar se perguntando, como acessar as instâncias que estão dentro de um subnet privada, já que não estão conectadas a internet?

Apresento a vocês o conceito do BASTION!

A idéia é uma máquina que está na Subnet pública, com Security Group público o qual tem acesso ao Security Group Privado. Nós vamos nos conectar ao Bastion público e fazer um "salto" a máquina privada.

## Criando os Security Groups

Antes de criar nosso Bastion e uma EC2 privada, vamos antecipar e criar dois Security Groups, um público e um privado.

-   Navegue até o menu Security Groups;
-   Clique em Create security group;
-   Preencha o nome, optei por `PublicSG-API`
-   Habilitei apenas SSH e HTTP.

Para o privado, seguir o mesmo procedimento, optei por chama-lo de `PrivateSG-API`

-   Além disso, nas regras de entrada (Inbound rules), permitir o `PublicSG-API` para SSH. Com essa permissão será possível fazer o salto do Bastion para a máquina privada.

## Criando o Bastion

-   Crie uma nova instância ec2, pode ser uma t3.micro, e não se esqueça de selecionar uma Subnet pública, `PublicSubnetA` por exemplo, e selecionar o Security Group público `PublicSG-API`.

## Criando a instância Privada

-   Crie uma nova instância ec2, pode ser uma t3.micro também, e não se esqueça de selecionar uma Subnet privada, `PrivateSubnetA` por exemplo, e selecionar o Security Group privado `PrivateSG-API`.

## O Salto!

Com tudo isso configurado, podemos tentar o salto ao Bastion e depois para a instância privada.

-   Faça SSH ao Bastion;
-   Copie o conteúdo da `key.pem` pra dentro do Bastion;
-   Faça SSH do Bastion a máquina privada.

> Bang! Deveria ter conseguido acessar a máquina privada que não tem acesso a internet!

# NAT Gateway

Se você seguiu tudo até aqui, significa que você está dentro da máquina privada. Tente fazer um `ping [google.com](<http://google.com>)` e veja o que acontece. Xablau! Não foi possível fazer o ping pois máquina privada não tem acesso a internet.

Para realizar tal feito, precisamos de um NAT Gateway.

## Criando um NAT Gateway

-   Navegue até a aba NAT Gateway, dentro de VPCs;
-   Clique em Create NAT Gateway;
-   Preencha o nome - segui a nomenclatura `nat-gw-us1-main`
-   Selecione a Subnet pública, selecionei a `PublicSubnetA`
-   Aloque um Elastic IP address;
-   Clique em create.

## Associando NAT Gateway ao Private Route Table

-   Navegue ate Route Table
-   Selecione a route table privada: `rtb-us1-main-private`
-   Clique em edit route;
-   Inclua uma rota com destination `0.0.0.0/0` e target `NAT Gateway` apontando pro recém criado `nat-gw-us1-main`

> That's IT! Se voltar para a instância privada e tentar novamente o `ping google.com`, verá que agora temos uma resposta: `64 bytes from [iad23s91-in-f14.1e100.net](<http://iad23s91-in-f14.1e100.net/>) (142.250.65.78): icmp_seq=4 ttl=109 time=1.94 ms`

> **Dica:** Se lembre que esse é apenas um NAT Gateway localizado na Zona A. O ideal é ter pelo menos mais um em outra Zona para ter maior disponibilidade.

# Uma abordagem mais barata: NAT Instancies