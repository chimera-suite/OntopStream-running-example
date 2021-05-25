## OntopStream running example

### 1. Start the producer (Kafka and Flink)

```
sudo docker-compose -f flink-kafka.yml up -d
```

### 2. Start the REST endpoint

Connectct to the `sql-client` remote terminal

```
sudo docker-compose -f flink-kafka.yml exec sql-client bash
```

Start the REST endpoint service

```
/opt/flink-sql-gateway-0.2-SNAPSHOT/bin/sql-gateway.sh --library /opt/sql-client/lib
```

__Note(1)__: keep the terminal window alive until you need the service. Close the REST service, press `CTRL+C` and type `EXIT` to leave the terminal.

__Note(2)__: the JDBC driver mappings are persisted on the local file [sql-gateway-defaults.yaml](./flink/flink-sql-gateway-0.2-SNAPSHOT/conf/sql-gateway-defaults.yaml), which is automatically loaded on startup in the `sql-client` docker image.

### 3. Start OntopStream

Open a new teminal window, then run the command:

```
sudo docker-compose -f ontop.yml up -d
```

### 4. Start Jupyter

```
sudo docker-compose -f jupyter.yml up
```

Load the notebook `OntopStream_user_notebook.ipynb` in the jupyter environment.

Run the examples inside the demo notebook.

### 5. Stop the demo environment

```
sudo docker-compose -f jupyter.yml down

sudo docker-compose -f ontop.yml down

sudo docker-compose -f flink-kafka.yml down
```

## Tools

Flink's built-in shell to run FlinkSQL queries to compare the results

```
sudo docker-compose -f flink-kafka.yml exec sql-client ./sql-client.sh
```
