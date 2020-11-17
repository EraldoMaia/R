#  R, trabalhando com a análise de dados abertos de viagens a serviço
 Analise e Tratamento de dados com a linguagem R

## Objetivo

A proposta deste tópico é colocar em prática algumas das funções do R trabalhando com a análise de dados abertos de viagens a serviço, com o intuito de subsidiar a tomada de medidas mais eficientes na redução dos gastos com os custos dessas viagens no setor público.

## Definição do problema

Para resolver um problema, primeiramente temos que entendê-lo. Assim, precisamos entender os gastos com viagens a serviço para tomar medidas mais eficientes e, com isso, reduzir os custos dessas viagens.

## Questões relevantes

Qual é o valor gasto com passagens por órgão?
Qual é o valor gasto com passagens por cidade?
Qual é a quantidade de viagens por mês?

## Obtenção dos dados

Primeiramente vamos acessar o portal da transparencia do governo federal. disponivel em: http://www.portaltransparencia.gov.br/
Na aba dados do portal, iremos extrair os dados de Viagens realizadas a serviço, referente ao ano de 2019.
Os dados extraidos estão no formato CSV.

## Carregando os dados

Acessando primeiramente a documentação da funcao que fará a leitura dos arquivos extraidos.
```{r}
?read.csv
```

Em seguida, aplicamos a função atribuindo a mesma em uma variavel chamada viagens.
```{r}
  viagens <- read.csv(file = "/Users/eraldomaia/Desktop/Enap/Análise de Dados em Linguagem R/Módulo 4 - Análise de Dados na Prática/AnaliseDeDadosViagensServico/Dados/Viagens-2019/2019_Viagem.csv", sep =';', dec = ',', fileEncoding = "latin1")
```


Agora verificamos se os dados foram carregados corretamente.
Essa funcao head() apresenta apenas as primeiras linhas do DataSet.
```{r}
head(viagens)
```

Com isso, iremosveridicar as dimensões do DataSet (Linhas x Colunas).

```{r}
dim(viagens)
```

Em seguida iremos recuperar algumas informações do DataSet, como o valor minimo, maximo e a média, referente as viagens a serviço.
Para isso, iremos utilizar a função summary().

Primeiramente, vamos acessar sua documentação.

```{r}
?summary
```

Em seguida, utilizamos a função no nosso DataSet.

```{r}
summary(viagens)

summary(viagens$Valor.passagens)
```

Agora, iremos verificar o tipo dos dados de cada coluna.
Para isso, iremos instalar o pacote "dplyr"

```{r}
 #install.packages("dplyr")
```

Em seguida carregamos o pacote instalado.

```{r}
library(dplyr)
```

Com o pacote "dplyr" carregado. Iremos utilizar a funcao "glimpse" para verificar o tipo dos dados de cada coluna.

```{r}
glimpse(viagens)
```

## Tratando os dados obtidos

Como podemos ver acima, as colunas "Período...Data.de.início" e "Período...Data.de.fim" estão como tipo "chr".
Nesse caso, será necessário transforma-las par tipo data.

Primeiramente, vamos acessar sua documentação.
```{r}
?as.Date
```

Em seguida, atribuimos a função na coluna "Período...Data.de.início" , para criar uma nova coluna, data.inicio com o tipo Data.
```{r}
viagens$data.inicio <- as.Date(viagens$Período...Data.de.início,"%d/%m/%Y")
```

A coluna original não foi alterada, apenas a nova "data.inicio".

Como podemos verificar abaixo.

```{r}
glimpse(viagens)
```

Agora formatamos o campo "data.inicio", para trabalhar apenas com o mes e o ano.

Utilizando a função "format()"

Primeiramente, vamos acessar sua documentação.

```{r}
?format
```

Em seguida formatamos o campo "data.inicio", criando um novo campo "data.inicio.mes.ano "

```{r}
viagens$data.inicio.mes.ano <- format(viagens$data.inicio,"%m-%y")
```

Verificando as alterações

```{r}
glimpse(viagens)
```

## Analise exploratória dos dados

Como iremos trbalhar na descobeta de custos com passagens, iremos fazer uma analise em cima dos dados referente ao valor da passagem.

```{r}
summary(viagens$Valor.passagens)
```

É possivel perceber que o valor maximo está bem distante da média, o que indica a presença de outliers. Que podem o resultado de uma analise que não reflita a realidade.

Sendo melhor visualizado em um grafico.
Nesse caso utilizaremos um grafico boxsplot.

```{r}
boxplot(viagens$Valor.passagens)
```
Além disso, iremos calcular o desvio padrao.

Utilizando a função "sd()"

Primeiramente, vamos acessar sua documentação.

```{r}
?sd
```

Em seguida calculamos o desvio padrao da coluna "Valor.passagens"

```{r}
sd(viagens$Valor.passagens)
```

Em seguida, vamos verificar se existem campos com valores não preechidos e contabilizar sua quantidade.

Utilizando a função "is.na()" e a função "colSums()"

Primeiramente, vamos acessar sua documentação.

```{r}
?is.na
?colSums
```

Agora verificamos se existem campos com valores não preechidos e contabilizar sua quantidade.

```{r}
colSums(is.na(viagens))
```


Como podemos ver cima, não existem campos que não foram preenchidos!

É interessante, a verificação da situação de cada viagem.

```{r}
table(viagens$Situação)
```

E a relação entre as viagens que foram realizadas e não realizadas

```{r}
sit <- prop.table(table(viagens$Situação))*100

sit
```

Temos que cerca de 97.7% das viagens pagas, foram realizadas. 

Vamos vizualizar em um grafico de pizza.

```{r}
?pie
pie(sit)
```
## Visualização dos resultados

A visualização dos resultados é a etapa final do nosso estudo acerca da análise dos dados de viagens a serviço.
Assim, é o momento de responder às questões levantadas na primeira fase: definição do problema.
Vamos relembrá-las? 

Qual é o valor gasto com passagens por órgão?
Qual é o valor gasto com passagens por cidade?
Qual é a quantidade de viagens por mês?


Primeiramente iremos descobrir quais orgãos estão gastando mais com passagens aereas.
Para isso iremos criar um "top 10" com os orgãos que estão gastando mais.

```{r}
p1 <- viagens %>% group_by(Nome.do.órgão.superior) %>% summarise(n = sum(Valor.passagens), .groups = 'drop') %>% arrange(desc(n)) %>% top_n(10)

names(p1) <- c("Orgão","Valor")

p1
```

Apos a consolidação dos dados, iremos plota-los em grafico.

```{r}
library(ggplot2)

ggplot(p1,aes(x=reorder(Orgão,Valor),y=Valor)) + geom_bar(stat = "identity", fill = "#9F1BF7") + geom_text(aes(label = Valor), vjust = 0.3, size = 3) + coord_flip() + labs(x="Valor",y= "Orgãos")
```
Agora iremos descobrir quais cidades estão gastando mais com passagens aereas.
Para isso iremos criar um "top 10" com as cidades que estão gastando mais.

```{r}
p2 <- viagens %>% group_by(Destinos) %>% summarise(n = sum(Valor.passagens), .groups = 'drop') %>% arrange(desc(n)) %>% top_n(10)

names(p2) <- c("Destino","Valor")

p2
```

Após a consolidação dos dados, iremos plota-los em grafico.   

```{r}
ggplot(p2,aes(x=reorder(Destino,Valor),y=Valor)) + geom_bar(stat = "identity", fill = "#EDAD1A") + geom_text(aes(label = Valor), vjust = 0.3, size = 3) + coord_flip() + labs(x="Valor",y= "Destino")
```

Agora iremos desobrir a quantidade de viagens realizadas por mês.

```{r}
p3 <-  viagens %>% group_by(data.inicio.mes.ano) %>% summarise(qtd = n_distinct(Identificador.do.processo.de.viagem), .groups = 'drop')

p3

```
Após a consolidação dos dados, iremos plota-los em grafico.  

```{r}
ggplot(p3, aes(x = data.inicio.mes.ano, y = qtd, group = 1)) + geom_line(colour = "#F74F1B") + geom_point() + geom_text(vjust = -1,aes(label = qtd))
```
