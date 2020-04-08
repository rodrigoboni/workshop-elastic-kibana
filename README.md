# workshop-elastic-kibana
Documentação, containers e exemplos de configuração e uso da stack Elastic

## Pré requisitos
* Git
* Docker
* Docker compose

## Inicialização e uso
* Iniciar containers elastic + kibana
    ```
    docker-compose up -d
    ```
* Visualizar / acompanhar logs
    ```
    docker-compose logs -f
    ```
* Estado dos containers:
    ```
    docker-compose ps
    ```
* Finalizar containers
    ```
    docker-compose down
    ```
* Acesso
    * [Elastic - http://localhost:9200](http://localhost:9200)
    * [Kibana - http://localhost:5601](http://localhost:5601)

## Referências
* [https://techlipe.github.io/Workshop-Zero-To-Hero/](https://techlipe.github.io/Workshop-Zero-To-Hero/)
* [https://www.meetup.com/pt-BR/Sao-Paulo-Elastic-Fantastics/events/269769295](https://www.meetup.com/pt-BR/Sao-Paulo-Elastic-Fantastics/events/269769295)
* [Dia 1](https://ela.st/br-day1)
* [Dia 2](https://ela.st/br-day2)
* [Dia 3](https://ela.st/br-day3)
* [Dia 4](https://ela.st/br-day4)
* [Dia 5](https://ela.st/br-day5)
