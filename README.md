# indicadoressaneamentomg
Projeto Indicadores de Saneamento Básico MG - FJP

---
title: "Dados SNIS"
author: "Joao Erick Costa"
date: "2024-12-02"
output: html_document
---
Definindo diretório


```{r}
setwd("C:/Users/Erick/OneDrive/Arquivos Computador(C)Joao/1-FJP/inputs/SNIS_data/agrupamento_dinamico")
```


Sobre os dados

O SNIS Municípios contém dados consolidados por município, abrangendo Água, Esgotos e Resíduos Sólidos. As informações estão organizadas em famílias de dados:

Água e Esgoto: Gerais, Contábeis, Operacionais, Financeiras, Qualidade, Tarifas, e PMSB.

Indicadores: Econômico-financeiros, administrativos, operacionais e de qualidade.

Resíduos Sólidos: Gerais, Coleta, Resíduos da construção civil, Coleta seletiva, Resíduos de serviços de saúde, Varrição, Capina e Unidades de processamento.

Os dados podem ser acessados de duas formas:

Informações e indicadores municipais consolidados: Totalizados para municípios atendidos por múltiplos prestadores.
Agrupamento dinâmico por ano de referência: Permite cálculos e agrupamentos de indicadores médios por município, região ou estado.
Ambos os métodos possibilitam filtros por ano, tipo de serviço, prestador e região.

Baixei os dados de agrupamento dinamico porque vem todas as variaveis do SNIS a nível de município. 


Quando baixa as planilhas no site do SNIS, o aquivo vem no formato csv e a planilha vem nomeada de forma aleatoria, como podemos observar: "AgrupamentoMunicipio-20241204172214" (essa planilha se refere ao ano 2000). Para faciliar a importação das planilhas aqui no R e para nao da conflitos no formato do arquivo, abri cada planilha de cada ano e salvei no formato xlsx e renomei o nome do arquivo pra ficar mais simples. Por exemplo, a planilha do ano 2000 recebeu o nome am2000, para o ano de 2001 o nome do arquivo ficou am2001 e assim sucessivamente ate 2022, que ficou am2022. Isso facilitará a importaçao das planilhas porque o R reconhece bem o formato xlsx e os nomes padronizados permite um trabalho fluido. 




Instalando pacotes e carregando

```{R}
install.packages("readxl") #Instala pacote (se tiver instalado nao precisa instalar)
install.packages("dplyr")
install.packages("writexl")

# Carregando pacotes
library(readxl)
library(dplyr)
library(writexl)


```


Importando arquivos e empilhando


```{r}

# Lista de arquivos (nomes das planilhas)
arquivos <- c("am2000.xlsx", "am2001.xlsx", "am2002.xlsx", "am2003.xlsx", "am2004.xlsx", "am2005.xlsx", "am2006.xlsx", "am2007.xlsx", "am2008.xlsx", "am2009.xlsx","am2010.xlsx","am2011.xlsx","am2012.xlsx","am2013.xlsx","am2014.xlsx","am2015.xlsx","am2016.xlsx","am2017.xlsx","am2018.xlsx","am2019.xlsx","am2020.xlsx","am2021.xlsx", "am2022.xlsx")

# Função para importar , além de padronizar os tipos de dados para caracteres 
importar_planilha <- function(arquivo) {
  # Extrai o ano do nome do arquivo
  ano <- as.numeric(gsub("\\D", "", arquivo))  #Extrai os números do nome do arquivo (representando o ano) utilizando a função gsub, que remove todos os caracteres que não sejam dígitos (\\D).
  
  
  
  # Lê a planilha
  dados <- read_excel(arquivo)   #Lê o arquivo Excel especificado no argumento arquivo, criando um data frame chamado dados.
  
# Converte todas as colunas para character (evita conflitos de tipos de dados nas colunas). Em alguns anos os operadores da base de dados SNIS podem ter salvado os dados de uma coluna de forma diferente e isso na hora de empilhar pode fazer com o R nao consiga compreender. Para resolver isso, basta indicar pro R que todas colunas serão character (é muito importante verificar cada coisa acontecendo e obsevar se os valores das variaveis nao estao se alterando nesse processo que estamos forçando). Aqui nos nossos dados é possivel verificar que os valores nao se alteram. Basta olhar os valores que vem nas planilhas originais e observar que nao acontecenada, entao podemos prosseguir. 
#Comando as baixo explicados: mutate(across(everything(), as.character)) Converte todas as colunas do data frame para o tipo character para evitar problemas de incompatibilidade de tipos ao combinar dados de diferentes planilhas.  mutate(Ano = ano): Adiciona uma nova coluna chamada Ano ao data frame, contendo o ano extraído anteriormente do nome do arquivo.

  dados <- dados %>%
    mutate(across(everything(), as.character)) %>%    
    mutate(Ano = ano)  # Adiciona a coluna do ano
  
  return(dados)
}  


#Explicação dos proximos comandos: dadossnis <- arquivos %>% lapply(importar_planilha) %>% bind_rows(): arquivos: É uma lista de nomes de arquivos Excel. lapply(importar_planilha): Aplica a função importar_planilha a cada arquivo da lista. Retorna uma lista de data frames. bind_rows(): Empilha (ou combina) os data frames da lista em um único data frame.

# Importar todas as planilhas e empilhar
dadossnis <- arquivos %>%
  lapply(importar_planilha) %>%
  bind_rows()

# Visualizar as primeiras linhas do data frame resultante
head(dadossnis)


#Removendo a primeira e as duas ultimas linhas do dataframe porque a planilha vem com informações totais (soma das variaveis) nessas linhas. Não nos interessa. slice(-1, -(n() - 1):-(n())): Remove: A primeira linha: -1. As duas últimas linhas: -(n() - 1):-(n()), onde: n() retorna o número total de linhas no data frame. n() - 1 representa a penúltima linha. n() representa a última linha. 

dadossnis <- dadossnis %>%
  slice(-1, -(n() - 1):-(n())) # Remove a primeira linha e as duas últimas



#Salvando dataframe dados SNIS (painel 2000-2022):
saveRDS(dadossnis, "dadossinis.rds")


```


Renomeando variaveis (facilitar manuseio dos dados) e atribuindo label


```{r}
library(dplyr)


dadossnis <- dadossnis %>%
  rename(
    regiao= `Região`,
    codigoibge = `Código do Município`,
    estado= `Estado`,
    ano = `Ano de Referência`, 
    prestadores= `Prestadores`, 
    servico = `Serviços`, 
    in001 = `IN001_AE - Densidade de economias de água por ligação`, 
    in002 = `IN002_AE - Índice de produtividade: economias ativas por pessoal próprio`, 
    in003= `IN003_AE - Despesa total com os serviços por m3 faturado`, 
    in004= `IN004_AE - Tarifa média praticada`, 
    in005= `IN005_AE - Tarifa média de água`,
    municipio = `Município`,
    g06a = `G06A - População urbana residente do(s) município(s) com abastecimento de água`,
    g06b = `G06B - População urbana residente do(s) município(s) com esgotamento sanitário`,
    g12a = `G12A - População total residente do(s) município(s) com abastecimento de água, segundo o IBGE`,
    g12b = `G12B - População total residente do(s) município(s) com esgotamento sanitário, segundo o IBGE`,
    poptotal = `POP_TOT - População total do município (Fonte: IBGE):`,
    popurb = `POP_URB - População urbana do município (Fonte: IBGE)`,
    in015 = `IN015_AE - Índice de coleta de esgoto`,
    in016 = `IN016_AE - Índice de tratamento de esgoto`,
    in20 = `IN020_AE - Extensão da rede de água por ligação`,
    in21 = `IN021_AE - Extensão da rede de esgoto por ligação`,
    in22 = `IN022_AE - Consumo médio percapita de água`,
    in23 = `IN023_AE - Índice de atendimento urbano de água`,
    in24 = `IN024_AE - Índice de atendimento urbano de esgoto referido aos municípios atendidos com água`,
    in25 = `IN025_AE - Volume de água disponibilizado por economia`,
    in46 = `IN046_AE - Índice de esgoto tratado referido à água consumida`,
    in47 = `IN047_AE - Índice de atendimento urbano de esgoto referido aos municípios atendidos com esgoto`,
    in52 = `IN052_AE - Índice de consumo de água`,
    in55 = `IN055_AE - Índice de atendimento total de água`,
    in16rs = `IN016_RS - Taxa de cobertura regular do serviço de coleta de rdo em relação à população urbana`
  )



#Adicionando atributos as variaveis

attr(dadossnis$in001, "label") <- "Densidade de economias de água por ligação"
attr(dadossnis$in002, "label") <- "Índice de produtividade: economias ativas por pessoal próprio"
attr(dadossnis$in003, "label") <- "Despesa total com os serviços por m3 faturado"
attr(dadossnis$in004, "label") <- "Tarifa média praticada"
attr(dadossnis$in005, "label") <- "Tarifa média de água"
attr(dadossnis$g06a, "label") <- "População urbana residente do(s) município(s) com abastecimento de água"
attr(dadossnis$g06b, "label") <- "População urbana residente do(s) município(s) com esgotamento sanitário"
attr(dadossnis$g12a, "label") <- "População total residente do(s) município(s) com abastecimento de água, segundo o IBGE"
attr(dadossnis$g12b, "label") <- "População total residente do(s) município(s) com esgotamento sanitário, segundo o IBGE"

attr(dadossnis$poptotal, "label") <- "População total do município (Fonte: IBGE)"
 
attr(dadossnis$popurb, "label") <- "População urbana do município (Fonte: IBGE)"

attr(dadossnis$in015, "label") <- "Índice de coleta de esgoto"
attr(dadossnis$in016, "label") <- "Índice de tratamento de esgoto"
attr(dadossnis$in20, "label") <- "Extensão da rede de água por ligação"
attr(dadossnis$in21, "label") <- "Extensão da rede de esgoto por ligação"
attr(dadossnis$in22, "label") <- "Consumo médio percapita de água"
attr(dadossnis$in23, "label") <- "Índice de atendimento urbano de água"
attr(dadossnis$in24, "label") <- "Índice de atendimento urbano de esgoto referido aos municípios atendidos com água"
attr(dadossnis$in25, "label") <- "Volume de água disponibilizado por economia"
attr(dadossnis$in46, "label") <- "Índice de esgoto tratado referido à água consumida"
attr(dadossnis$in47, "label") <- "Índice de atendimento urbano de esgoto referido aos municípios atendidos com esgoto"
attr(dadossnis$in52, "label") <- "Índice de consumo de água"
attr(dadossnis$in55, "label") <- "Índice de atendimento total de água"
attr(dadossnis$in16rs, "label") <- "Taxa de cobertura regular do serviço de coleta de rdo em relação à população urbana"


```



Nesse estágio da manipulação dos dados temos 661 variáveis. Vou filtrar apenas as variaveis que nos interessa para melhor manuseio dos dados. As variaveis retiradas do banco de dados podem ser obtidas no bando "dadossnis" (caso um dia precise voltar atras). O novo dataframe se chamará dadossnisfiltrado. 

```{r}
# Selecionando as colunas desejadas
dadossnisfiltrado <- dadossnis %>%
  select(
    estado,
    codigoibge,
    regiao,
    ano,
    prestadores,
    servico,
    municipio,
    g06a,
    g06b,
    g12a,
    g12b,
    poptotal,
    popurb,
    in015,
    in016,
    in20,
    in21,
    in22,
    in23,
    in24,
    in25,
    in46,
    in47,
    in52,
    in55,
    in16rs
  )

```



Os dados estao no formato de characters desde o inicio para nao da conflito na hora de empilhar os dados. Agora vamos atribuir as variaveis o verdadeiro formato da variavel. Quem é character será character e quem é numeric será numeric. 

```{r}
library(readr)
library(dplyr)

dadossnisfiltrado <- dadossnisfiltrado %>%
  mutate(
    regiao = as.character(regiao),
    codigoibge = parse_number(codigoibge, locale = locale(decimal_mark = ",")),  # Valor com vírgula
    estado = as.character(estado),
    ano = parse_number(ano, locale = locale(decimal_mark = ",")),  # Valor com vírgula
    prestadores = as.character(prestadores),
    servico = as.character(servico),
    municipio = as.character(municipio),
    g06a = parse_number(g06a),  # Valor com ponto
    g06b = parse_number(g06b),  # Valor com ponto
    g12a = parse_number(g12a),  # Valor com ponto
    g12b = parse_number(g12b),  # Valor com ponto
    poptotal = parse_number(poptotal),  # Valor com ponto
    popurb = parse_number(popurb),  # Valor com ponto
    in015 = parse_number(in015, locale = locale(decimal_mark = ",")),  # Valor com vírgula
    in016 = parse_number(in016, locale = locale(decimal_mark = ",")),  # Valor com vírgula
    in20 = parse_number(in20, locale = locale(decimal_mark = ",")),  # Valor com vírgula
    in21 = parse_number(in21, locale = locale(decimal_mark = ",")),  # Valor com vírgula
    in22 = parse_number(in22, locale = locale(decimal_mark = ",")),  # Valor com vírgula
    in23 = parse_number(in23, locale = locale(decimal_mark = ",")),  # Valor com vírgula
    in24 = parse_number(in24, locale = locale(decimal_mark = ",")),  # Valor com vírgula
    in25 = parse_number(in25, locale = locale(decimal_mark = ",")),  # Valor com vírgula
    in46 = parse_number(in46, locale = locale(decimal_mark = ",")),  # Valor com vírgula
    in47 = parse_number(in47, locale = locale(decimal_mark = ",")),  # Valor com vírgula
    in52 = parse_number(in52, locale = locale(decimal_mark = ",")),  # Valor com vírgula
    in55 = parse_number(in55, locale = locale(decimal_mark = ",")),  # Valor com vírgula
    in16rs = parse_number(in16rs, locale = locale(decimal_mark = ","))  # Valor com vírgula
  )


```






Quando atribuimos o formato numeric a variavel codigoibge, o R entendeu que existem NA na variavel. Com isso verifiquei quem sao eles e realmente nao possui informaçoes para eles, inclusive, eles estavam fazendo a base de dados ficar com 19685 observações, quando na verdade deviaria ficar 19619, porque 23(anos)x853(municipios)=19619. Entao os NA eram quem estavam fazendo a base de mais informações. 

```{r}
# Verificar valores não numéricos na coluna 'codigoibge'
valores_nao_numericos <- dadossnisfiltrado %>%
  filter(!grepl("^\\d+$", codigoibge)) %>%  # Mantém apenas valores que não são totalmente numéricos
  select(codigoibge) %>%
  distinct()

# Exibir os valores problemáticos
print(valores_nao_numericos)

#Removendo NA
dadossnisfiltrado <- dadossnisfiltrado %>%
  filter(grepl("^\\d+$", codigoibge))


```






Verificando se há dados duplicados



```{r}
# Verificar o número de anos e municípios únicos
num_anos <- length(unique(dadossnisfiltrado$ano))
num_municipios <- length(unique(dadossnisfiltrado$codigoibge))

cat("Número de anos:", num_anos, "\n")
cat("Número de municípios:", num_municipios, "\n")

# Verificar o número total de combinações municipio-ano
num_combinacoes <- nrow(dadossnisfiltrado %>%
  distinct(codigoibge, ano))

cat("Número total de combinações municipio-ano:", num_combinacoes, "\n")

# Verificar se o painel está balanceado
total_esperado <- num_anos * num_municipios
if (num_combinacoes == total_esperado) {
  cat("O painel está balanceado.\n")
} else {
  cat("O painel não está balanceado.\n")
}



```


Verificando dados duplicados

```{r}
# Verificar se há duplicatas
duplicados <- dadossnisfiltrado %>%
  filter(duplicated(paste(municipio, ano)))

if (nrow(duplicados) > 0) {
  cat("Existem dados duplicados.\n")
} else {
  cat("Não há dados duplicados.\n")
}


```

Atribuindo label a base de dados filtrados:


```{r}
attr(dadossnisfiltrado$g06a, "label") <- "População urbana residente do(s) município(s) com abastecimento de água"
attr(dadossnisfiltrado$g06b, "label") <- "População urbana residente do(s) município(s) com esgotamento sanitário"
attr(dadossnisfiltrado$g12a, "label") <- "População total residente do(s) município(s) com abastecimento de água, segundo o IBGE"
attr(dadossnisfiltrado$g12b, "label") <- "População total residente do(s) município(s) com esgotamento sanitário, segundo o IBGE"

attr(dadossnisfiltrado$poptotal, "label") <- "População total do município (Fonte: IBGE)"
 
attr(dadossnisfiltrado$popurb, "label") <- "População urbana do município (Fonte: IBGE)"

attr(dadossnisfiltrado$in015, "label") <- "Índice de coleta de esgoto"
attr(dadossnisfiltrado$in016, "label") <- "Índice de tratamento de esgoto"
attr(dadossnisfiltrado$in20, "label") <- "Extensão da rede de água por ligação"
attr(dadossnisfiltrado$in21, "label") <- "Extensão da rede de esgoto por ligação"
attr(dadossnisfiltrado$in22, "label") <- "Consumo médio percapita de água"
attr(dadossnisfiltrado$in23, "label") <- "Índice de atendimento urbano de água"
attr(dadossnisfiltrado$in24, "label") <- "Índice de atendimento urbano de esgoto referido aos municípios atendidos com água"
attr(dadossnisfiltrado$in25, "label") <- "Volume de água disponibilizado por economia"
attr(dadossnisfiltrado$in46, "label") <- "Índice de esgoto tratado referido à água consumida"
attr(dadossnisfiltrado$in47, "label") <- "Índice de atendimento urbano de esgoto referido aos municípios atendidos com esgoto"
attr(dadossnisfiltrado$in52, "label") <- "Índice de consumo de água"
attr(dadossnisfiltrado$in55, "label") <- "Índice de atendimento total de água"
attr(dadossnisfiltrado$in16rs, "label") <- "Taxa de cobertura regular do serviço de coleta de rdo em relação à população urbana"

```


Salvando base de dados filtrada:

```{r}
#Salvando dataframe dados SNIS (painel 2000-2022):
saveRDS(dadossnis, "dadossinisfiltrado.rds")

```


Obtendo estatisticas descritivas da variável 


```{r}
library(dplyr)
library(tidyr)

estatisticas <- dadossnisfiltrado %>%
  summarise(
    in23_min = min(in23, na.rm = TRUE),
    in23_mean = mean(in23, na.rm = TRUE),
    in23_max = max(in23, na.rm = TRUE),
    in24_min = min(in24, na.rm = TRUE),
    in24_mean = mean(in24, na.rm = TRUE),
    in24_max = max(in24, na.rm = TRUE),
    in47_min = min(in47, na.rm = TRUE),
    in47_mean = mean(in47, na.rm = TRUE),
    in47_max = max(in47, na.rm = TRUE),
    in16rs_min = min(in16rs, na.rm = TRUE),
    in16rs_mean = mean(in16rs, na.rm = TRUE),
    in16rs_max = max(in16rs, na.rm = TRUE)
  ) %>%
  pivot_longer(
    cols = everything(),  # Transformar todas as colunas em uma tabela longa
    names_to = c("variable", "statistic"),  # Dividir os nomes das colunas
    names_sep = "_"  # Separador que identifica "variável" e "estatística"
  ) %>%
  pivot_wider(
    names_from = variable,  # Transformar as variáveis em colunas
    values_from = value  # Usar os valores como conteúdo das células
  )


#atribuindo label
attr(estatisticas$in23, "label") <- "Índice de atendimento urbano de água"
attr(estatisticas$in24, "label") <- "Índice de atendimento urbano de esgoto referido aos municípios atendidos com água"
attr(estatisticas$in47, "label") <- "Índice de atendimento urbano de esgoto referido aos municípios atendidos com esgoto"
attr(estatisticas$in16rs, "label") <- "Taxa de cobertura regular do serviço de coleta de rdo em relação à população urbana"
```



Grafico de nuvem dos dados em cada ano das variaveis de agua, esgoto e residuos:


```{r}
library(ggplot2)
library(dplyr)

# Filtra os dados para remover NA's
dados_filtrados <- dadossnisfiltrado %>%
  filter(!is.na(in23))  # Remover NAs

# Criando gráficos para cada ano
grafico_in23_ano <- dados_filtrados %>%
  ggplot(aes(x = factor(municipio), y = in23)) +  # 'municipio' como variável categórica
  geom_point(alpha = 0.7, aes(color = as.factor(ano))) +  # Adiciona pontos coloridos por ano
  facet_wrap(~ano, scales = "free_x") +  # Cria uma faceta para cada ano
  labs(
    title = "Índice de atendimento urbano de água por Ano (2000-2022)",
    x = "Município",
    y = "Índice",
    color = "Ano"
  ) +
  theme_minimal() +
  theme(
    axis.text.x = element_blank(),  # Remove os rótulos do eixo X (não exibe os municípios)
    axis.ticks.x = element_blank(),  # Remove as marcas do eixo X
    legend.position = "none"  # Remove a legenda
  )

# Exibir o gráfico
print(grafico_in23_ano)



```
