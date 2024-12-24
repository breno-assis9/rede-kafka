# Rede-Kafka

## Descrição

**Rede-Kafka** é uma aplicação desenvolvida em **.NET 6** para integrar sistemas distribuídos via **Apache Kafka**. Este projeto tem como objetivo fornecer uma solução robusta e escalável para o processamento de mensagens em tempo real, utilizando o poder da plataforma .NET e as capacidades do Kafka.

## Tecnologias

- **.NET 6** (C#)
- **Apache Kafka** (Kafka Producer & Consumer)
- **Docker** (Para orquestrar containers do Kafka)
- **Serilog** (Para logging)
- **Kafka .NET Client** (Confluent.Kafka)
- **Docker Compose** (Para facilitar o setup do Kafka e Zookeeper)

## Funcionalidades

- **Produção de Mensagens**: Envia mensagens para tópicos Kafka.
- **Consumo de Mensagens**: Consome mensagens em tempo real de tópicos Kafka.
- **Escalabilidade**: A aplicação pode ser facilmente escalada para trabalhar com múltiplos produtores e consumidores.
- **Monitoramento e Logging**: Logs detalhados da aplicação para monitoramento, usando o Serilog.
- **Facilidade de Configuração**: Usando Docker Compose para configurar Kafka e Zookeeper sem necessidade de instalação manual.

## Requisitos

- **.NET 6 SDK** ou superior.
- **Apache Kafka** e **Zookeeper** em funcionamento.
- **Docker** (opcional, para executar Kafka e Zookeeper em containers).

## Instalação

### 1. Clonando o Repositório

```
git clone https://github.com/usuario/rede-kafka.git
cd rede-kafka

```

### 2. Configuração do Kafka com Docker
Caso não tenha o Kafka instalado localmente, você pode usar Docker para configurar rapidamente o ambiente.

Crie um arquivo docker-compose.yml com a seguinte configuração:

 ``` 
version: '3'

services:
  zookeeper:
    image: wurstmeister/zookeeper:3.4.6
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka:
    image: wurstmeister/kafka:latest
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENER: INSIDE_KAFKA:9093
      KAFKA_LISTENER_SECURITY_PROTOCOL: PLAINTEXT
      KAFKA_LISTENER_NAMES: INSIDE
      KAFKA_LISTENER_INSIDE_PORT: 9093
      KAFKA_LISTENER_INSIDE_HOSTNAME: kafka
      KAFKA_LISTENER_OUTSIDE_PORT: 9092
    ports:
      - "9092:9092"
      - "9093:9093"
    depends_on:
      - zookeeper
```

Para iniciar os containers, execute:

docker-compose up -d

### 3. Configuração do Projeto

Após o ambiente de Kafka estar em funcionamento, você pode configurar as variáveis de ambiente para os detalhes de conexão. Adicione as configurações no arquivo appsettings.json:

```
{
  "Kafka": {
    "BootstrapServers": "localhost:9092",
    "TopicName": "rede-kafka-topic"
  }
}
```

### 4. Execução do Projeto

Execute o projeto usando o seguinte comando:

```
dotnet run
O serviço começará a produzir e consumir mensagens a partir do Kafka.
```

### Endpoints

**1. Produzir Mensagem**

A aplicação pode ser configurada para enviar mensagens para o Kafka. Você pode chamar o endpoint HTTP para produzir mensagens:

```
POST /api/kafka/produce
Body (JSON):
```
```
{
  "message": "Olá, Kafka!"
}
```

** 2. Consumir Mensagem **

O consumidor Kafka irá escutar as mensagens em tempo real. Você pode configurar a aplicação para processar mensagens diretamente da fila.

Exemplo de Código
Aqui está um exemplo básico de um produtor e consumidor Kafka em .NET Core 6.

Kafka Producer (Produtor de Mensagens)
```
public class KafkaProducer
{
    private readonly IProducer<Null, string> _producer;

    public KafkaProducer(IConfiguration config)
    {
        var producerConfig = new ProducerConfig { BootstrapServers = config["Kafka:BootstrapServers"] };
        _producer = new ProducerBuilder<Null, string>(producerConfig).Build();
    }

    public async Task SendMessageAsync(string message)
    {
        await _producer.ProduceAsync("rede-kafka-topic", new Message<Null, string> { Value = message });
    }
}
```

Kafka Consumer (Consumidor de Mensagens)

```
public class KafkaConsumer
{
    private readonly IConsumer<Null, string> _consumer;

    public KafkaConsumer(IConfiguration config)
    {
        var consumerConfig = new ConsumerConfig
        {
            BootstrapServers = config["Kafka:BootstrapServers"],
            GroupId = "rede-kafka-consumer",
            AutoOffsetReset = AutoOffsetReset.Earliest
        };

        _consumer = new ConsumerBuilder<Null, string>(consumerConfig).Build();
        _consumer.Subscribe("rede-kafka-topic");
    }

    public void StartConsuming()
    {
        while (true)
        {
            var consumeResult = _consumer.Consume();
            Console.WriteLine($"Mensagem recebida: {consumeResult.Message.Value}");
        }
    }
}

```

### Contribuição

Contribuições são bem-vindas! Se você deseja contribuir com este projeto, siga os passos abaixo:

Faça um fork deste repositório.
Crie uma branch para sua feature (git checkout -b minha-feature).
Commit suas alterações (git commit -am 'Adiciona nova feature').
Push para a branch (git push origin minha-feature).
Abra um pull request.

### Licença
Este projeto está licenciado sob a MIT License - veja o arquivo LICENSE para mais detalhes.




