# Relatório de Análise de Dados - Big Data

## Introdução

Este projeto realiza uma análise de dados de diversos canais do YouTube voltados para ciência de dados e análise. Utilizando PySpark, um poderoso framework para processamento de grandes volumes de dados, a análise visa extrair insights sobre o desempenho dos vídeos desses canais. As principais métricas analisadas incluem o número de visualizações, a contagem de likes e o número de comentários.

Acesse o arquivo original em:<br>
https://colab.research.google.com/drive/1L5Kuhm-KctxGJw0lqi9UM4aPnFWHobXX
<br>

Caso não consiga acessar, por qualquer motivo, replique a metodologia seguindo as instruções.

## Instruções

### 1. Crie um Novo Notebook no Google Colab

### 2. Instale o Pyspark:
```bash
    !pip install pyspark
```

### 3. Importe os dados necessários direto do Kaggle:
```bash
    from google.colab import files
    files.upload()  # Import o arquivo kaggle.json disponibilizado no repositório
```
 - Esse arquivo kaggle.json contém as minhas credencias para fazer a conexão com a API da Kaggle logo mais. Fique a vontade para usar nesse projeto.

### 4. Depois de fazer o upload, configure a API do Kaggle:
```bash
    !mkdir -p ~/.kaggle
    !cp kaggle.json ~/.kaggle/
    !chmod 600 ~/.kaggle/kaggle.json

    # Baixando o conjunto de dados do Kaggle:
    !kaggle datasets download -d abhishek0032/youtube-dataset-all-data-scienceanalyst-channels
    !unzip youtube-dataset-all-data-scienceanalyst-channels.zip
```

### 5. Agora, vamos carregar os dados no Pyspark:
```bash
    from pyspark.sql import SparkSession

    # Iniciando uma sessão Spark
    spark = SparkSession.builder.appName("YouTubeDataAnalysis").getOrCreate()

    # Carregando o conjunto de dados
    df = spark.read.csv("data.csv", header=True, inferSchema=True)
    df.show(5)
```
### 6. Limpeza simples nos dados para evitar incongruências:
```bash
    df.printSchema()

    df = df.dropDuplicates()

    # Remover linhas com valores nulos em colunas importantes
    df = df.na.drop(subset=["Channel_Name", "Title", "Published_date", "Views", "Like_count", "Comment_Count"])

    df.show(5)
```

### 7. Converter colunas em type numérico:
```bash
    from pyspark.sql.functions import col

    # Função que achei para converter colunas para inteiros
    df = df.withColumn("Views", col("Views").cast("int"))
    df = df.withColumn("Like_count", col("Like_count").cast("int"))
    df = df.withColumn("Comment_Count", col("Comment_Count").cast("int"))

    df.printSchema()
```
  - Converter em int type irá possibilitar realizar a média dos valores de forma simples com avg().

### 8. Análise dos dados:
```bash
    # Contar o número total de canais
    num_canais = df.select("Channel_Name").distinct().count()
    print(f"Número total de canais: {num_canais}")

    # Contar o número de vídeos por canal
    videos_por_canal = df.groupBy("Channel_Name").count().withColumnRenamed("count", "Num_videos")
    videos_por_canal.show(5)

    # Cálculo da média de visualizações por canal
    media_visualizacoes_por_canal = df.groupBy("Channel_Name").avg("Views").withColumnRenamed("avg(Views)", "Media_Views")
    media_visualizacoes_por_canal.show(5)

    # Cálculo da média de likes por canal
    media_likes_por_canal = df.groupBy("Channel_Name").avg("Like_count").withColumnRenamed("avg(Like_count)", "Media_Likes")
    media_likes_por_canal.show(5)

    # Cálculo da média de comentários por canal
    media_comentarios_por_canal = df.groupBy("Channel_Name").avg("Comment_Count").withColumnRenamed("avg(Comment_Count)", "Media_Comentarios")
    media_comentarios_por_canal.show(5)
```
### 9. Exporte os resultados da análise dos dados para fazer o relatório:
```bash
    videos_por_canal.write.csv("/mnt/data/videos_por_canal.csv", header=True)
    media_visualizacoes_por_canal.write.csv("/mnt/data/media_visualizacoes_por_canal.csv", header=True)
    media_likes_por_canal.write.csv("/mnt/data/media_likes_por_canal.csv", header=True)
    media_comentarios_por_canal.write.csv("/mnt/data/media_comentarios_por_canal.csv", header=True)
```
  - Localize os arquivos seguindo o caminho que foram salvos no Google Colab e faça download ou simplesmente os baixe na pasta ".csv" disponibilizada no repositório.

<br>

# Realizado por:
### Yago Raímon Xavier Cavalcanti Silva
