# workshop-elastic-kibana

Resumo do workshop sobre Elastic promovido pelo grupo Elastic São Paulo, entre 06/04/2020 e 10/04/2020

## Pré requisitos p/ executar Elastic + Kibana local (via Docker)

- Git
- Docker
- Docker compose

## Inicialização e uso dos containers

- Iniciar containers elastic + kibana
  ```
  docker-compose up -d
  ```
- Visualizar / acompanhar logs
  ```
  docker-compose logs -f
  ```
- Estado dos containers:
  ```
  docker-compose ps
  ```
- Finalizar containers
  ```
  docker-compose down
  ```
- Acesso
  - [Elastic - http://localhost:9200](http://localhost:9200)
  - [Kibana - http://localhost:5601](http://localhost:5601)

## Dia 1 - Apresentação Elastic Stack

- [Transmissão](https://ela.st/br-day1)
- Beats e Logstash
  - Serviços de ingestão de dados / coleta de dados (em hosts, containers, aplicações etc)
  - Agentes que coletam dados
  - Podem modificar / processar os dados antes de enviar (ETL)
  - Envia dados ao Elastic
- Elastic
  - Não é apenas uma ferramenta de monitoração (ex Grafana)
  - Engine de coleta e armazenamento de dados
  - Engine de busca de dados planejada para:
    - Escalabilidade
    - Performance
    - Relevância
- Kibana
  - Interface de visualização de dados e gerenciamento do Elastic

## Dia 2 - Observabilidade

- [Transmissão](https://ela.st/br-day2)

- Acessar a home do Kibana p/ ver lista de módulos de observabilidade disponíveis, com instruções de instalação e configuração - [home](http://localhost:5601/app/kibana#/home)

- Beats
  - P/ configurar os módulos ajustar arquivos yml na pasta /etc/metricbeat/modules.d
  - Em ambiente de produção seria necessário configurar o cluster elastic p/ onde os dados devem ser enviados (em ambiente local é automático)
  - Metricbeat
    - Coletar e enviar métricas do SO ao Elastic
    - Habilitar módulo docker (e outros seguindo a mesma base):
      ```
      sudo metricbeat modules enable docker
      ```
    - Configurar e iniciar os módulos:
      ```
      sudo metricbeat setup
      sudo service metricbeat start
      ```
    - No Kibana procurr pelos dashboards "Metricbeat docker" e "Metricbeat system"
      ![GitHub Logo](/images/systemmetrics.png)
      ![GitHub Logo](/images/metricdocker.png)
  - Filebeat
    - Coletar e enviar dados de logs
    - Logs de qualquer app / serviço rodando no SO
    - Padroniza o formato de log que é enviado ao Elastic / permite ajustes
    - Habilitar coleta de logs do SO e iniciar serviço:
      ```
      sudo filebeat modules enable system
      sudo filebeat setup
      sudo service filebeat start
      ```
    - No Kibana procurar pelo dashboard "Filebeat system"
      ![GitHub Logo](/images/filebeat.png)
  - Auditbeat
    - Praticamente a mesma coleta do Filebeat, porém focada em auditoria de segurança (logins, ip de origem, tentativas de login com sucesso e falha)
    - Habilitar coleta:
      ```
      sudo auditbeat setup
      sudo service auditbeat start
      ```
    - No Kibana procurar pelo item "SIEM"
      ![GitHub Logo](/images/SIEM.png)

- APM - Application Performance Monitoring
  - Monitorar performance de uma aplicação através da coleta de dados específicos da app
  - Composto por:
    - Client - roda junto da aplicação, geralmente como uma dependência
    - Server - roda em um servidor, recebe os dados do client e envia ao Elastic
  - Mesmo conceito / ideia do New Relic (monitoração de transações web / Data Insights)
  - [O que é o Application Performance Monitoring (APM)?](https://www.elastic.co/pt/blog/monitoring-applications-with-elasticsearch-and-elastic-apm)

## Dia 3 - Conhecendo o Elasticsearch

- [Transmissão](https://ela.st/br-day3)

- É um mecânismo de busca / base de dados não relacional / schema less
- Estrutura de dados baseada em JSON
- Permite buscas escaláveis / buscas rápidas mesmo em bases com muitos dados

- O Elastic é composto por clusters com seus respectivos nós (ao rodar o container local estamos subindo um nó)

  - Dados do nó (usar a opção "Dev tools" do Kibana):

    ```
    GET /

    {
    "name" : "e3617abde627", (nome do nó)
    "cluster_name" : "docker-cluster", (nome do cluster)
    "cluster_uuid" : "JnJggtUFR16lNwstxUfl1g",
    "version" : {
        "number" : "7.6.1",
        "build_flavor" : "default",
        "build_type" : "docker",
        "build_hash" : "aa751e09be0a5072e8570670309b1f12348f023b",
        "build_date" : "2020-02-29T00:15:25.529771Z",
        "build_snapshot" : false,
        "lucene_version" : "8.4.0",
        "minimum_wire_compatibility_version" : "6.8.0",
        "minimum_index_compatibility_version" : "6.0.0-beta1"
    },
    "tagline" : "You Know, for Search"
    }
    ```

  - Nós em um cluster:

    ```
    GET _cat/nodes?v

    ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
    172.18.0.2           23          90  31    2.71    2.81     2.84 dilm      *      951c6a15bc56

    ```

- O Elastic trabalha com duas camadas de comunicação, uma HTTP e outra de transporte (TCP)
  - A camada http rest é utilizada para responder requests
  - A camada de transporte é utilizada para comunicação entre os nós do cluster
    ![GitHub Logo](/images/elasticsearch-fluxo.png)

- Tipos de nós

  - Data node
    - Armazena / indexa dados
    - Retorna consultas
    - Agregação
  - Master node
    - Coordena nós
    - Integridade do cluster
    - Distribuição dos dados entre os nós
    - Distribui Shards
    - Federação dos nós
    - Recomendado a partir de 7 nodes segregar em mais de um master
  - Ingest note
    - Processa pipelines de ingestão (enriquece / modifica dados antes de persistir)
  - Coordinator node
    - Distribui requisições entre os data nodes (load balancer?)
  - Machine learning node
    - Processa jobs e funções de machine learning

- Dados no elastic:
    - Índice
        - Mapeamento / estrutura de dados p/ organizar e indexar as pesquisas
        - Pode ser feito de forma automática ou manual (dependendo do caso é aconselhado mapear manualmente)
        - Adicionar documento e criar índice / mapear automáticamente (qdo o índice não existir)

        - clientes = nome do índice
        - \_doc = tipo do índice (padrão)
        - /1 = id do documento

            ```
            PUT clientes/_doc/1
            {
            "nome" :  "Zé Mockinho",
            "rua"  : "Av. sei lá aonde",
            "numero": "123",
            "idade" : "22"
            }
            ```

        - Para ter id automático, utilize POST:
            ```
            POST clientes/_doc
            {
            "nome" :  "Zé Mockinho",
            "rua"  : "Av. sei lá aonde",
            "numero": "123",
            "idade" : "22"
            }
            ```

        - Criar índice:
        ```
            PUT clientes/
            {
            "mappings": {
                "properties": {
                "nome" : {
                    "type" : "text",
                    "fields": {
                    "keyword" : {
                        "type" : "keyword"
                    }
                    }
                },
                "rua" : {
                    "type" : "keyword"
                },
                "numero" : {
                    "type": "integer"
                },
                "idade" : {
                    "type" : "integer"  (obs tipo de dados)
                }
                }
            }
            }
        ```

    - Shards e réplicas
        - Menor unidade de armazenamento / indexação
        - Um documento de um índice é armazenado e indexado em um Shard, que é armazenado em disco
        - Os Shards são replicados entre os nós p/ otimizar as buscas (shards primárias e réplicas)
        - Junto com os índices são os principais pontos a serem tratados quanto a performance / otimização de um cluster Elastic

    - Comandos de pesquisa e manipulação:
        - Ver mapeamento definido para um índice:
            ```
            GET clientes/_mapping
            ```
        - Pesquisar
            ```
            GET clientes/_search
            {
            "query": {
                "match": {
                "nome": "Felipe"
                }
            }
            }
            ```
        - Remover índice
            ```
            DELETE clientes
            ```
        - Estado do cluster
            ```
            GET _cluster/health

            {
            "cluster_name" : "docker-cluster",
            "status" : "yellow",
            "timed_out" : false,
            "number_of_nodes" : 1,
            "number_of_data_nodes" : 1,
            "active_primary_shards" : 6,
            "active_shards" : 6,
            "relocating_shards" : 0,
            "initializing_shards" : 0,
            "unassigned_shards" : 4,
            "delayed_unassigned_shards" : 0,
            "number_of_pending_tasks" : 0,
            "number_of_in_flight_fetch" : 0,
            "task_max_waiting_in_queue_millis" : 0,
            "active_shards_percent_as_number" : 60.0
            }
            ```

            - O estado "yellow" indica que o cluster não conseguiu indexar uma shard réplica
            - O estado "red" indica que não foi possível indexar uma shard primária

            ```
            GET _cluster/allocation/explain

            {
            "index" : "clientes",
            "shard" : 0,
            "primary" : false,
            "current_state" : "unassigned",
            "unassigned_info" : {
                "reason" : "CLUSTER_RECOVERED",
                "at" : "2020-04-12T23:30:06.047Z",
                "last_allocation_status" : "no_attempt"
            },
            "can_allocate" : "no",
            "allocate_explanation" : "cannot allocate because allocation is not permitted to any of the nodes",
            "node_allocation_decisions" : [
                {
                "node_id" : "RaVAePGJRpS75taA-SFXuA",
                "node_name" : "951c6a15bc56",
                "transport_address" : "172.18.0.2:9300",
                "node_attributes" : {
                    "ml.machine_memory" : "8259338240",
                    "xpack.installed" : "true",
                    "ml.max_open_jobs" : "20"
                },
                "node_decision" : "no",
                "deciders" : [
                    {
                    "decider" : "same_shard",
                    "decision" : "NO",
                    "explanation" : "the shard cannot be allocated to the same node on which a copy of the shard already exists [[clientes][0], node[RaVAePGJRpS75taA-SFXuA], [P], s[STARTED], a[id=rwC-4ztpRWCIdpFyReKoGw]]"
                    }
                ]
                }
            ]
            }
            ```

## Dia 4 -

- [Transmissão](https://ela.st/br-day4)




## Dia 5 -

- [Transmissão](https://ela.st/br-day5)





## Referências

- [https://www.meetup.com/pt-BR/Sao-Paulo-Elastic-Fantastics/events/269769295](https://www.meetup.com/pt-BR/Sao-Paulo-Elastic-Fantastics/events/269769295)
- [https://techlipe.github.io/Workshop-Zero-To-Hero/](https://techlipe.github.io/Workshop-Zero-To-Hero/)
- [https://medium.com/@fqueirooz80](https://medium.com/@fqueirooz80)
