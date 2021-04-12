# Deploy RabbitMQ
    docker run -d --hostname my-rabbit --name some-rabbit rabbitmq:3
# Check IP address 
    docker network inspect bridge
# Create Producer, Dockerfile
    FROM python:3-alpine
    RUN mkdir -p /usr/src/app
    WORKDIR /usr/src/app
    # Install the PIKA library
    RUN pip3 install pika
    # This variable forces pika to print something out
    ENV PYTHONUNBUFFERED=1
    COPY producer.py /usr/src/app
    ENTRYPOINT ["python3"]
    CMD ["producer.py"]

# Producer Code
    import pika

    # Connect to RabbitMQ
    connection = pika.BlockingConnection(pika.ConnectionParameters('172.17.0.5'))
    channel = connection.channel()

    # Create a queue
    channel.queue_declare(queue='hello’)

    # Send the message
    channel.basic_publish(exchange='', routing_key='hello', body='Hello World!')

    # Close the channel
    connection.close()

# Consumer Code
    import pika

    # Connect to RabbitMQ
    connection = pika.BlockingConnection(pika.ConnectionParameters('172.17.0.5'))
    channel = connection.channel()

    # Create a queue
    channel.queue_declare(queue='hello')

    # Send the message
    channel.basic_publish(exchange='', routing_key='hello', body='Hello World!')

    # Close the channel
    connection.close()
    
# Build and run
    docker build -t rabbitmq-consumer .
    docker build -t rabbitmq-producer .

    docker run -d --name consumer rabbitmq-consumer
    docker run -d --name producer rabbitmq-producer

    docker logs consumer
    docker logs producer
    docker logs rabbitqm
    
    
# Exchange example
    # Message producer
    …
    # There is no need for queue declaration (I use the exchange)
    channel.exchange_declare(exchange=‘exchange_name', exchange_type='direct')

    channel.basic_publish(exchange='exchange_name', routing_key='name1', body='hello1')
    channel.basic_publish(exchange='exchange_name', routing_key='name2', body='hello2')

    connection.close()

    # Message consumer
    …
    channel.exchange_declare(exchange='exchange_name', exchange_type='direct')

    # I let the system to create the queue name
    result = channel.queue_declare(queue='', exclusive=True)
    queue_name = result.method.queue

    # Bind the queue to one or more keys/exchanges (it can be done at runtime)
    channel.queue_bind(exchange='exchange_name', queue=queue_name, routing_key='name1')
    channel.queue_bind(exchange='exchange_name', queue=queue_name, routing_key='name2')

    channel.basic_consume(
        queue=queue_name, on_message_callback=callback, auto_ack=True)
    channel.start_consuming()
    
# Topic example
    # Producer
    channel.exchange_declare(exchange='topics', exchange_type='topic')

    routing_key1 = 'lazy.green.rabbit'
    routing_key2 = 'fast.yellow.zebra'

    message = 'Hello World!'
    channel.basic_publish(exchange='topics', routing_key=routing_key1, body=message)
    channel.basic_publish(exchange='topics', routing_key=routing_key2, body=message)

    connection.close()

    # Consumer
    channel.exchange_declare(exchange='topics', exchange_type='topic')

    result = channel.queue_declare('', exclusive=True)
    queue_name = result.method.queue

    channel.queue_bind(exchange='topics', queue=queue_name, routing_key='*.*.rabbit')
    channel.queue_bind(exchange='topics', queue=queue_name, routing_key='lazy.#')

# Check RabbitMQ status
    docker exec ID_CONTAINER rabbitmqctl list_queues
    docker exec ID_CONTAINER rabbitmqctl list_queues name messages_ready messages_unacknowledged





