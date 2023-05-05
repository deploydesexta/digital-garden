
# Amazon Aurora: Design Considerations for High Throughput Cloud-Native Relational Databases

# Introdução

Amazon Aurora é um banco de dados relacional criado para OLTP oferecido como parte do ecossistema da AWS. Nos sistemas em nuvem modernos, escalabilidade e resiliência é mais facilmente atingido quando separamos computação de armazenamento e replicação de armazenamento entre múltiplos nodes. Essa arquitetura permite facilmente substituir um node com problemas por uma outra réplica sem precisa se preocupar com a escritura e processamento.

O gargalo de I/O encontrado na maioria dos sistemas tradicionais não acontece nesse ambiente. Já que I/O agora pode ser dividido para múltiplos nodes e discos, o gargalo de um disco individual deixa de ser problema, movendo para a rede que interliga a camada de database a qual solicita o processamento I/O da camada de armazenamento que de fato performa o I/O.

Amazon Aurora é possível por conta de ser um serviço de banco de dados que delega a complexidade para uma camada de _redo log_ altamente distribuído na nuvem.

# Referência

- [https://web.stanford.edu/class/cs245/readings/aurora.pdf](https://web.stanford.edu/class/cs245/readings/aurora.pdf)