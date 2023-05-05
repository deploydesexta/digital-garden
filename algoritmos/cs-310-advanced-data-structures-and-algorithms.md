# CS-310 - Advanced Data Structures and Algorithms


# **Lecture 14 - Distributed DB Consistency**

## **Recapitulando sobre Base de Dados NoSQL**

- Particionamento de dados é necessário para dividir a carga de escrita para cada Node.
- Base de Dados NoSQL torna particionamento mais fácil por não armazenar referências.
- Sem referências entre os dados, é necessário clonar a informação e a informação fica denormalizada.
- Bases NoSQL distribuídas são MUITO escaláveis mas são limitados a ser *key-value*.
    - A chave é indexada.
- Distributed Hash Tables podem implementar uma base NoSQL.
    - As chaves hashs são divididas igualmente entre os storages nodes.
    - O client que determina a chave que determina o node.
- Uma arquitetura Shared Nothing é a distribuição independente de cada Node.
- Cada request é tratada por **um** node.
- Tanto throughput e capacidade são diretamente proporcionais a quantidade de Nodes.
- Hash Partitioning (a.k.a Distributed Hash Tables - DHTs) podem ser escalados para milhares de Nodes.

## **Mas o que acontece com *reliability*?**

- Quanto mais nodes, maior a chance de falhas. Para resolver esse problema precisamos replicar a informação.
- É criado um *overlap* nos hashs ranges dos nodes.
    
    ![replicação](/assets/cs-310-advanced-data-structures-and-algorithms/range-replication.png)
    

### **Consistência**

- O problema quando os dados são replicados é que quando acontece uma escritura o dado precisa ser replicado para todos os nodes, e isso pode causar uma inconsistência caso aconteça uma leitura no meio tempo da atualização de todas as réplicas.
    
    ![consistency](/assets/cs-310-advanced-data-structures-and-algorithms/consistency.png)
    

### **CAP Theorem**

 O teorema mais famoso dos sistemas distribuídos!

Ele diz que qualquer sistema distribuídos não podem ter as três propriedades a seguir, você precisa escolher apenas duas.

**C**onsistency

Significa que todas as leituras vão receber o dados mais atualizado ou receberão um erro.

**A**vailability

Toda request não receberá um erro e sim um dado independente se é o mais recente ou não.

**P**artition Tolerance

O conteúdo de algumas partições podem ser excluídas ou sofrerem algum delay.

**Ou seja...**

- Em uma base de dados distribuída, quando os nodes não estão em sincronia, nós devemos aceitar **inconsistência** ou esperar os nodes sincronizarem.
- Para conseguir com que uma base distribuída responda imediatamente uma resposta consistente, precisamos que a rede seja 100% confiável e sem delay.
- O teorema de CAP nos dá o *trade-off* entre **consistência** e **delay.**
- Inconsistência é algo que causa muitos bugs, já delay é algo que podemos lidar.

→ **Se realmente precisamos de consistência e zero-delay então vá para uma base relacional.**

### **NoSQL fornece algumas maneiras de lidar com consistência e delay**

**Monotonic Reads**

**Read your Writes**

**Monotonic Writes**

### Duas maneiras de atingir Consistência Eventual

- O client sempre envia requests para um mesmo node (sticky session)
    - Pró: Muito simples de implementar
    - Con: Problema de consistência acontece quando um node falha
        - MongoDB suporta Consistência e Tolerancia a Particionamento. Não replica as informações e se um node cair, haverá downtime até backup (C-P).
- O client esperar para que tudo (leitura e escritura) estejam sincronizados entre todos nodes.
    - **Como saber se uma chave está sincronizada?**
        - O cliente enviar a request para todos os nodes e esperar o sucesso de tudo.

### Esperar por consistência com Quorums

- Essa é uma das formas de atingir consistência eventual.
- Um Quorum é uma porcentagem minima de aceites.
- Espera ate que uma certa quantidade de nodes enviem um OK para garantir que a leitura, ou escritura, deu certo.
- Previne o progresso até que certa quantidade de nodes responda com OK.
- Enviamos a request para todos os nodes e esperamos N respostas.

| Quorum de escritura | Quorum de leitura | Otimizado para |
| --- | --- | --- |
| Todos os nodes | Apenas um | Leituras rápidas |
| Maioria | Maioria | Balanceado em leitura e escritura |
| Apenas um | Todos os nodes | Escrituras rápidas |


### Como isso é escalável?

- No exemplo temos três replicas em três nodes e isso faz com que todos os nodes tenham todos os dados - Isso não escala.
- Normalmente, haverá uma quantidade N de nodes para uma quantidade X de réplicas.
- No cenário real, pode haver 10 nodes para uma 3 de réplicas.

![cluster](/assets/cs-310-advanced-data-structures-and-algorithms/cluster-scaling.png)

# Referências

[https://www.youtube.com/watch?v=Vpq6TUY7v6I](https://www.youtube.com/watch?v=Vpq6TUY7v6I)
[https://edisciplinas.usp.br/pluginfile.php/2541318/mod_resource/content/1/TeoremaDeBrewer.pdf](https://edisciplinas.usp.br/pluginfile.php/2541318/mod_resource/content/1/TeoremaDeBrewer.pdf)