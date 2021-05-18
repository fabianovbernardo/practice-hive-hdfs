## TRABALHANDO COM HIVE

Esse repositório tem como objetivo demonstrar uma prática com o framework Hive em conjunto com HDFS.
Lembrando que essa prática foi utilizando o docker!

#### LEVANTAR OS CLUSTERS
Para estartar os clusters, vamos executar o segunte comando dentro do repositório de Big Data
#docker-compose up -d

1-Imagem

#### ETAPA 1: DESLOCANDO O ARQUIVO LOCAL PARA O HDFS
Antes de manipular qualquer tipo de tabela vamos acessar e utilizar o beeline, que é um cliente Hive que está incluindo nos principais clusters.

2-Imagem

#### ETAPA 2: CRIAÇÃO DE UMA TABELA INTERNA
> Criar a Tabela Hive no BD empresastartup - TABELA INTERNA: pop com os campos:

-   zip_code - int
-   total_population - int
-   median_age - float
-   total_males - int
-   total_females - int
-   total_households - int
-   average_household_size - float

E as propriedades:
-   Delimitadores: Campo ‘,’ | Linha ‘\n’
-   Sem Partição
-   Tipo do arquivo: Texto
-   tblproperties("skip.header.line.count"="1")

Agora vamos criar uma tabela interna utilizando o hive com uma consulta SQL `create table empresastartup.pop(zip_code int, total_population inr, median_age float, total_males int, total_females int, total_households int, average_household_size float) row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile tblproperties("skip.header.line.count"="1");`

3 - imagem

 Agora podemos consultar essa tabela nas imagens abaixo, podendo observar a sua estrutura comos seus tipos de dados! Na segunda imagem aplicando o comando `desc formatted empresastartup.pop;`, e é possível observar o caminho que foi gravado.

4 - imagem

5- imagem


ETAPA 3: INSERÇÃO DE DADOS NA TABELA

 Antes de mais nada vamos listar os primeiros 10 registros da tabela pop! Como é possível observar não foi encontrado nenhum registro.
 
6 - imagem
 
 Carregar o arquivo do HDFS “/user/aluno//data/população/populacaoLA.csv” para a tabela Hive pop

O arquivo já está no HDFS, uma questão interessante é mapear o diretório /user/aluno/gabriel/data/populacao/. Por que? Porque o nosso arquivo pode estar em vários "pedaços". Mas neste caso vamos mapear o arquivo de forma direta.

7 - imagem
 Questão: Contar a quantidade de registros da tabela pop

Pronto, agora aplicando uma consulta SQL  `select * from pop limit 10;`  é possível observar que agora a nossa tabela pop tem registro!

Realizando a última questão podemos contar a quantidade de registros da tabela pop com a seguinte consulta  `select count (*) from pop;`

8 - imagem

ETAPA 4: CRIAÇÃO DE UMA TABELA EXTERNA E TRABALHANDO COM SELEÇÃO DE TABELAS

Para trabalhar com a seleção de tabelas, vamos criar uma pasta no namenode e posteriormente realizar uma consulta do diretório  `$ docker exec -it namenode hdfs dfs -mkdir /user/aluno/gabriel/data/nascimento`. Na figura abaixo é possível observar os diretórios já criados!

8 - imagem


E agora vamos verificar se a base de dados e tabela está presente! Verificar se a base de dados as tabelas estão presentes é processo importante quando trabalhamos com dados.

Quando trabalhamos com hive é possível criar dois tipos de tabelas: Tabelas Internas e Tabelas Externas. Quando trabalhamos com Tabelas Externas, o Hive não move os dados para seu diretório de armazém. Se a tabela externa for descartada, os metadados da tabela serão excluídos, mas não os dados. Agora, se trabalhamos com tabelas internas, o Hive move os dados para seu diretório de armazém. Se a tabela for eliminada, os metadados da tabela e os dados serão excluídos.Para criar a tabela externa nascimento foi necessario aplicar o seguinte comando  `create external table nascimento(nome string, sexo string, frequencia int) partitioned by (ano int) row format delimited fields terminared by ',' lines terminated by '\n' stored as textfile location '/user/aluno/gabriel/data/nascimento';`

9 - imagem



Após a criação da tabela externa, vamos adicionar na partição a tabela nascimento do ano de 2015. Na segunda imagem é possível observar que temos um arquivo para cada ano, por exemplo: yob1999.

10 - imagem
11- imagem

Observando esse ponto vamos mover da nossa pasta para a nossa partição utilizando o comando `hdfs dfs -put /input/exercises-data/names/yob2015.txt /user/aluno/gabriel/nascimento/ano=2015`

12 - imagem
13- imagem

E agora vamos verificar se a nossa partição tem registros com a consulta `select * from nascimento limit 20`

14 - imagem

> Questão: Selecionar os 5 primeiros registros da tabela nascimento pelo ano de 2016;

Utilizando a consulta  `select * from nascimento where ano=2015 limit 5`  é possível observar os 5 primeiros registros do ano de 2015.

15 - imagem


> Questão: Contar a quantidade de nomes de crianças nascidas em 2017 e contar a quantidade de crianças nascidas em 2017

Agora vamos contar a quantidade de nomes com a consulta  `select count(nome) as qtd from nascimento where ano=2017;`  e o sistema irá retornar a quantidade de nomes de crianças do ano de 2017! Vale lembrar que, quando aplicamos consultas sobre uma base de dados ou partição, o Hive realiza a consulta como um MapReduce.

Depois de saber a quantidade de nomes no ano de 2017 vamos contar a quantidade de crianças nascidas em 2017 com o comando  `select sum(frequencia) as qtd from nascimento where ano=2017;`

16 - imagem


> Questão: Contar a quantidade de crianças nascidas por sexo no ano de 2015 e mostrar por ordem de ano decrescente a quantidade de crianças nascidas por sexo

Para realizar esse processo vamos contar a quantidade de crianças nascidas por sexo no ano de 2015 com a consulta  `select sexo, sum(frequencia) as qtd from nascimento where ano=2015 group by sexo;`  e assim o sistema irá retornar a quantidade de crianças que nasceram em 2015 filtrado por sexo.

Agora vamos mostrar por ordem de ano descrecente a quantidade de crianças nascidas por sexo, e para isso vamos consultar com o comando  `select ano, sexo, sum(frequencia) as qtd from nascimento group by ano, sexo order by ano desc;`

17- imagem

> Questão: Contar a quantidade de crianças nascidas por sexo no ano de 2015 e mostrar por ordem de ano decrescente a quantidade de crianças nascidas por sexo

Para realizar esse processo vamos contar a quantidade de crianças nascidas por sexo no ano de 2015 com a consulta  `select sexo, sum(frequencia) as qtd from nascimento where ano=2015 group by sexo;`  e assim o sistema irá retornar a quantidade de crianças que nasceram em 2015 filtrado por sexo.

Agora vamos mostrar por ordem de ano descrecente a quantidade de crianças nascidas por sexo, e para isso vamos consultar com o comando  `select ano, sexo, sum(frequencia) as qtd from nascimento group by ano, sexo order by ano desc;`

18-imagem

> Questão: Mostrar por ordem de ano decrescente a quantidade de crianças nascidas por sexo com o nome iniciado com ‘A’ e qual nome e quantidade das 5 crianças mais nascidas em 2016

Para resolver a primeira questão vamos mostrar por ordem de ano decrecente, a quantidade de crianças por sexo com o nome iniciado com 'A'. Para isso vamos aplicar a seguinte consulta  `select ano, sexo, sum(frequencia) as qtd from nascimento where nome like 'A%' group by ano, sexo order by ano desc;`  e só assim o sistema irá retornar a quantidade de crianças com a letra 'A' nascidas de acordo com cada ano.

Na segunda questão, vamos verificar qual o nome e quantidade das 5 crianças mais nascidas em 2016 com o comando  `select nome, max (frequencia) as qtd from nascimento where ano=2016 group by nome order by qtd desc limit 5;`
19-imagem

> Questão: Qual nome e quantidade das 5 crianças mais nascidas em 2016 do sexo masculino e feminino

Agora para resolver a última questão, queremos saber qual o nome e quantidade das 5 crianças mais nascidas em 2016 do sexo masculino e feminino, e para isso vamos utilizar a seguinte consulta  `select nome, max(frequencia) as qtd, sexo from nascimento where ano=2016 group by nome, sexo order by qtd desc limit 5;`

20-imagem

