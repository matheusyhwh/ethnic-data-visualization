# Visualização de dados de alta dimensão utilizando R

## Datasets utilizados
Os datasets utilizados foram os spreadsheets das calculadoras do GEDmatch: Eurogenes K13 e MDLP 22.
As tabelas foram pre-processadas antes de serem utilizadas. Para fins de estudo, resolvemos focar nas etnias judaicas e nos grupos que tiveram contato cultural mais substancial com os grupos judaicos. Removemos etnias que não estavam dentro do escopo do trabalho. Neste reositório você encontra as tabelas intocadas (K13full e K22full) e também as processadas (K3limpo e K22limpo). Fazendo os devidos ajustes, sinta-se livre para usar outras tabelas.

## PCA, t-SNE e UMAP
O script abaixo gera o PCA de um .csv que esteja devidamente rotulado. Por exemplo: no caso do dataset Eurogenes, lemos o .csv e o atribuímos para a variável seuDataset. Depois, definimos que os rótulos serão a primeira coluna do dataset. Eles serão necessários, pois precisamos ver os nomes das etnias, e não somente os pontos no gráfico. Em seguida, atualizamos o  seuDataset excluindo a primeira coluna dele (usada para os rótulos) ao selecionar apenas a partir da segunda coluna até n, onde n é o número da última coluna. No caso do Eurogenes, o n será 14 e no MDLP 22, será 23. 
``` R
seuDataset <- read.csv(" local do arquivo ”)
rownames(seuDataset) <- seuDataset[,1] 
seuDataset <- seuDataset[,2:n]
```
É indispensável a instalação de todos os pacotes antes do uso.  Caso ainda não possua os pacotes necessários (ggfortify, Rtsne e umap), basta ir em Pacotes > Instalar Pacotes e procurar por eles e realizar a instalação, e em seguida rodar o código abaixo. 
```R
library(ggfortify)
pca_res <- prcomp(seuDataset, scale.=TRUE)
 ``` 
Chamamos o pacote ggortify, e realizamos a análise dos componentes principais utilizando o prcomp. É possível omitir o scale. = TRUE caso você não queira realizar a normalização dos dados. Após isso, plotamos o gráfico com base na lista de valores retornada por prcomp
	
```R
autoplot(pca_res, x = 1, y = 2, label = TRUE, label.size = 3)
```
ou
```R
pcaplot<-ggplot(df_pca,aes(x=PC1,y=PC2,label = rownames(seuDataset) ))+geom_text(size = 3)
print(pcaplot)
```
O pacote prcomp realiza a análise de Componentes Principais nos dados fornecidos. O valor retornado é usado para a construção do gráfico através do pacote ggplot2 ou da função genérica autoplot. O gráfico terá duas dimensões. Os outros parâmetros fazem com que seja utilizado o rótulo das amostras ao invés do ponto, e o tamanho da fonte do rótulo é setado. Por fim, o software apresentará a tela com os rótulos dispostos com suas respectivas distâncias.
Para realizar a plotagem do t-SNE, utiliza-se o script:
```R
library(Rtsne)
tsne <- Rtsne(seuDataset, check_duplicates=FALSE, perplexity = 89)
df_tsne <- data.frame(x = tsne$Y[,1], y = tsne$Y[,2])
tsneplot<-ggplot(df_tsne, aes(x, y, label = rownames(etnias))) + geom_text(size = 3)
print(tsneplot)
```
O pacote Rtsne realiza a plotagem do t-SNE no dataset passado como parâmetro. Caso ocorram valores duplicados ou duas ou mais amostras de uma mesma etnia, pode-se opcionalmente  removê-las sentando check_duplicates para TRUE. Em termos práticos, a perplexidade diz respeito ao modo de agrupamentos: quanto menor for a perplexidade, mais ilhas com menor quantidade de componentes haverão; e quanto maior, mais dispersa a representação será. Além da perplexidade, podemos realizar diversas outras alterações, como o o número máximo de iterações, e o valor de theta, que diz respeito ao trade-off entre velocidade e acurácia, sendo 0 o valor do t-SNE original, com maior acurácia, e 1 o valor para maior rapidez e menos acurácia (sendo o valor default 0.5). Utilizamos todos os valores default, e testamos uma perplexidade alta e uma baixa.
Por fim, o UMAP: Chamamos o pacote umap, utilizamos o método default Naive, preparamos o dataframe e plotamos, semelhantemente ao realizado nos métodos anteriores.

```R
library(umap)
umap <- umap(seuDataset, method="naive")
df <- data.frame(x = umap$layout[,1], y = umap$layout[,2])
ggplot(df, aes(x, y, label = rownames(seuDataset))) + geom_text(size = 3)
```
## microbenchmark
Aplicamos o pacote Microbenchmark para avaliar os três métodos. Segundo a documentação do R, O Microbenchmark é um pacote de funções de temporização precisas, e “fornece infraestrutura para medir e comparar com precisão o tempo de execução de expressões R.
A utilização do microbenchmark é simples:. 
```R
library(ggplot2)
library(ggfortify)
library(microbenchmark)
mbm <- microbenchmark(
          "Nome do método" = {
[aqui vai o código de seu método]
          },
          "Nome de outro método" = { 
[aqui vai o código de seu outro método]
          },
[...]
               check = NULL)
	    
```

Para realizar esta análise, o código abaixo foi utilizado:
```R
library(ggplot2)
library(ggfortify)
library(microbenchmark)
mbm <- microbenchmark(
          "UMAP MDLP" = {
            library(umap)
            umap <- umap(seuDataset, method="naive")
            df <- data.frame(x = umap$layout[,1], y = umap$layout[,2])
            umapplot <- ggplot(df, aes(x, y, label = rownames(seuDataset))) + geom_text(size = 3)
            print(umapplot)
          },
          "t-SNE MDLP p3" = {
            library(Rtsne)
            tsne <- Rtsne(seuDataset, perplexity = 34)
            df <- data.frame(x = tsne$Y[,1], y = tsne$Y[,2])
            tsneplot <- ggplot(df, aes(x, y, label = rownames(seuDataset))) + geom_text(size = 3)
	    print(tsneplot)
          },
          "t-SNE MDLP p34" = {
            library(Rtsne)
            tsne34 <- Rtsne(seuDataset, perplexity = 34)
            df34 <- data.frame(x = tsne34$Y[,1], y = tsne34$Y[,2])
            tsneplot34 <- ggplot(df34, aes(x, y, label = rownames(seuDataset))) + geom_text(size = 3)
	    print(tsneplot34)
          },

          "PCA MDLP" = {
            df_pca <- prcomp(seuDataset)
            pcaplot <- ggplot(df_pca,aes(x=PC1,y=PC2,label = rownames(seuDataset) ))+geom_text(size = 3)
            print(pcaplot)
          },
          "UMAP Eurogenes" = {
            library(umap)
            umap2 <- umap(seuDataset2, method="naive")
            df2 <- data.frame(x = umap2$layout[,1], y = umap2$layout[,2])
            umapplot2 <- ggplot(df2, aes(x, y, label = rownames(seuDataset2))) + geom_text(size = 3)
            print(umapplot2)
          },
          "t-SNE Eurogenes p3" = {
            library(Rtsne)
            tsne22 <- Rtsne(seuDataset2, perplexity = 18)
            df22 <- data.frame(x = tsne22$Y[,1], y = tsne22$Y[,2])
            tsneplot22 <- ggplot(df22, aes(x, y, label = rownames(seuDataset2))) + geom_text(size = 3)
	    print(tsneplot22)
          },
          "t-SNE Eurogenes p18" = {
            library(Rtsne)
            tsne2 <- Rtsne(seuDataset2, perplexity = 18)
            df2 <- data.frame(x = tsne2$Y[,1], y = tsne2$Y[,2])
            tsneplot2 <- ggplot(df2, aes(x, y, label = rownames(seuDataset2))) + geom_text(size = 3)
	    print(tsneplot2)
          },
          "PCA Eurogenes" = {
            df_pca2 <- prcomp(seuDataset2)
            pcaplot2 <- ggplot(df_pca2,aes(x=PC1,y=PC2,label = rownames(seuDataset2) ))+geom_text(size = 3)
            print(pcaplot2)
          },
               check = NULL)
```
Iniciamos chamando os pacotes necessários para a análise. Depois, chamamos o microbenchmark, passando como parâmetro os diversos códigos aos quais desejamos comparar - quatro situações foram testadas em cada um dos datasets: PCA, t-SNE com perplexidade alta (34 no MDLP, 18 no Eurogenes), t-SNE com perplexidade baixa (3), UMAP.  
Para ver a lista de valores coletados, como (média mediana, menor valor, maior valor, etc):
```R
mbm
```
Para ver o gráfico de violino:
```R
autoplot(mbm)
```

No caso de quaisquer dúvidas ou sugestões, sinta-se livre para contactar-me pelo e-mail matheusyhwh@gmail.com
