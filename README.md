# Relatório Airbnb NYC | 2011–2019

## Problema de negócio

O Airbnb é uma plataforma digital que conecta anfitriões que desejam alugar seus imóveis a hóspedes que buscam acomodações temporárias. Na cidade de Nova York,  considerado um dos mercados mais relevantes da plataforma, milhares de anúncios estão distribuídos entre diferentes distritos, tipos de acomodação e faixas de preço.
Com a expansão da plataforma, o Airbnb passou a gerar um grande volume de dados relacionados à oferta de anúncios, preços praticados, receita estimada, avaliações dos hóspedes, níveis de engajamento e perfil dos anfitriões. Apesar dessa abundância de dados, a análise estratégica dessas informações não é trivial quando elas não estão organizadas de forma estruturada e orientada a negócio.  
<br>
**Do ponto de vista executivo, existe a necessidade de responder perguntas-chave como:** <br />  
- Onde a oferta de anúncios está mais concentrada?  
- Quais distritos e bairros concentram maior potencial de receita?  
- Como o engajamento dos hóspedes se distribui pela cidade?  
- Qual é o perfil dos anfitriões: ocasionais ou profissionais?
<br>  
Sem uma visão consolidada dos principais KPIs estratégicos, torna-se difícil identificar padrões, concentrações, desequilíbrios territoriais e oportunidades de otimização da plataforma. Antes da aplicação de modelos preditivos ou análises avançadas, a principal necessidade do projeto é a criação de um dashboard executivo, capaz de centralizar os principais KPIs de oferta, receita, engajamento e anfitriões, permitindo uma leitura clara, intuitiva e orientada à tomada de decisão.  
<br />
<br>
<br />

## Premissas assumidas para a análise  
Para a construção das análises e visualizações deste projeto, foram adotadas as seguintes premissas, considerando as limitações e características do dataset:  
- O dataset representa um recorte histórico compreendido entre 2011 e 2019, não refletindo variações sazonais detalhadas nem mudanças posteriores no mercado.
- Os registros disponíveis representam anúncios ativos na plataforma, sendo essa a unidade básica utilizada nas análises de oferta, preço, engajamento e anfitriões.
- As informações de preço representam o valor anunciado por noite e são utilizadas como referência para análises de posicionamento e potencial de receita, não correspondendo necessariamente ao valor efetivamente pago em todas as reservas.
- A receita estimada é calculada com base em regras de negócio definidas no projeto e deve ser interpretada exclusivamente para fins analíticos e comparativos.
- O número de avaliações é tratado como um indicador indireto de engajamento dos hóspedes, assumindo que anúncios mais avaliados tendem a apresentar maior nível de interação na plataforma.
- Cada anfitrião é identificado de forma única por meio do host_id, assumindo que esse identificador representa corretamente um único anfitrião na plataforma.
<br>
<br />  

## Etapa 1 – Preparação dos Dados e Regras de negócios
Nesta etapa foi realizada a ingestão, limpeza, padronização e enriquecimento do dataset AB_NYC_2019, utilizando Power Query e uma modelagem em camadas, com o objetivo de garantir consistência, escalabilidade e clareza analítica.  
O projeto foi estruturado em múltiplas consultas independentes (consulta base, analítica, IQR, calendário, localização, anfitriões e medidas), cada uma com uma responsabilidade bem definida. Essa abordagem evita referências cíclicas e facilita a manutenção, reutilização e evolução do pipeline de dados.  
<br>
<br />

### Consulta: AB_NYC_2019_base
A consulta AB_NYC_2019_base representa a camada de dados tratados e padronizados, sem aplicação de regras analíticas ou de negócio, servindo como fundação confiável para as demais etapas.  

<br> **Principais atividades realizadas:**  <br />
- Importação do arquivo CSV original
- Promoção de cabeçalhos e tipagem correta das colunas
- Remoção de colunas não utilizadas na análise (review_per_month, calculated_host_listing_count, availability_365)
- Limpeza e padronização de textos (Trim, Clean e Proper)
- Tratamento de erros e remoção de registros inválidos
- Remoção de registros com valores nulos em colunas essenciais (id, neighbourhood, neighbourhood_group, latitude, longitude, room_type e price)
- Ajustes e validações de latitude e longitude por meio de lógica recursiva, garantindo consistência geoespacial

<br>Foi antecipado o cálculo da coluna final_price (preço por noite × noites mínimas) para a camada base. Embora se trate de uma regra analítica, essa decisão foi tomada por necessidade técnica, a fim de evitar dependência circular entre a consulta de detecção de outliers (IQR) e a camada analítica. Dessa forma, garantiu-se a estabilidade do pipeline e a independência entre as etapas de transformação.<br />
<br>
<br />

### Consulta: AB_NYC_2019_análise
A consulta AB_NYC_2019_análise é construída como referência direta da AB_NYC_2019_base e concentra exclusivamente regras analíticas, flags de negócio e colunas derivadas, respeitando a separação de responsabilidades.  

<br> **Principais regras aplicadas:**  <br />
- Classificação dos anúncios em categorias de preço (price_category), com base na coluna final_price
- Criação da flag de anúncios com / sem avaliação (review_status)
- Classificação de registros como normais ou atípicos, com base nos limites calculados pelo IQR (price_outlier e final_price_outlier)
- Definição do status de atividade do anúncio com base na data de last_review
- Classificação das avaliações em faixas de engajamento (review_range)
- Criação de índices numéricos para categorias de preço, status de atividade e faixas de engajamento, facilitando ordenações e análises visuais

<br>Essa camada representa o núcleo analítico do projeto, preparando os dados para consumo direto em métricas, KPIs e visualizações.  <br />
<br>
<br />

### Consulta IQR_price e IQR_final_price (Detecção de Atípicos)
A identificação de outliers foi realizada por meio do método estatístico IQR (Interquartile Range), em consultas separadas, evitando dependências circulares no modelo.  

<br> **Etapas do processo:**  <br />
- Cálculo do primeiro quartil (Q1) e do terceiro quartil (Q3)
- Cálculo do intervalo interquartil (IQR)
- Definição dos limites inferior e superior para identificação de valores atípicos

<br>Essa abordagem permite análises com e sem outliers, de acordo com o objetivo analítico de cada visualização.<br />
<br>
<br />

### Consulta: Calendário
Foi criada uma tabela calendário, baseada na menor e maior data presente em last_review.

<br> **A tabela contém:**  <br />
- Data
- Dia da Semana, Dia do Mês e Dia do Ano
- Mês e Nome do Mês
- Ano
- Trimestre e Nome do Trimestre
- Semestre e Nome do Semestre
- Semana do Mês e Semana do Ano
- Coluna Ano/Mês com índice numérico para ordenação  
<br>
<br />

### Consulta: Localização  
Foi criada uma dimensão de Localização a partir das colunas neighbourhood e neighbourhood_group, contendo:
- Latitude e longitude médias
- Geração de um identificador único (location_id)

<br>Essa dimensão permite análises territoriais consistentes e contribui para a organização e desempenho do modelo.<br />
<br>
<br />

### Consulta: Anfitriões  
Foi criada a dimensão Anfitriões, responsável por consolidar informações no nível de anfitrião, utilizando host_id como chave primária.

<br> **Principais regras aplicadas:**  <br />
- Identificação única de anfitriões (host_id e host_name)
- Cálculo do total de anúncios por anfitrião (total_ads)
- Criação da faixa de anúncios (ads_range) e do índice (ads_range_index)  

Essa dimensão sustenta análises de concentração e profissionalização dos anfitriões, sendo fundamental para a Visão de Anfitriões do dashboard.
<br>
<br />

### Consulta: Medidas
Todas as medidas DAX foram centralizadas em uma tabela de Medidas, organizada em pastas temáticas:
- Oferta
- Preço
- Localização
- Avaliações
- Anfitriões

<br>Essa organização melhora a governança, manutenção e legibilidade do modelo.<br />
<br>
<br />

### Modelagem de Dados  
O modelo de dados segue um padrão estrela simplificado, no qual:
- A tabela Analítica atua como tabela fato
- As dimensões de Calendário, Localização e Anfitriões se relacionam à fato por meio de chaves bem definidas

<br>Essa estrutura garante performance, clareza analítica e escalabilidade.<br />
<br>
<br />

### Conclusão da Etapa
Esta etapa demonstra conceitos fundamentais de gestão e preparação de dados, separação de responsabilidades, criação de dimensões, tratamento estatístico e organização de pipelines analíticos, com aplicação prática em Power Query, preparando uma base sólida para análises e visualizações.
<br>
<br />

## Etapa 2 - Análise e Dashboard
Nesta etapa, os dados preparados e modelados na Etapa 1 foram utilizados para a construção de um dashboard executivo em Power BI, para realização de análise estratégica. As análises foram organizadas em quatro visões de negócio, cada uma respondendo a perguntas específicas sobre o mercado de Airbnb em Nova York.

### Visão Oferta (Anúncios)
Esta visão tem como objetivo analisar a distribuição, concentração e localização da oferta de anúncios do Airbnb.

<br> **Principais análises:** <br />
- Quantidade total de anúncios por distrito e bairro
- Oferta média por bairro, utilizada como métrica de normalização da concentração territorial
- Identificação de distritos e bairros responsáveis pela maior parcela da oferta (Pareto 80/20)
- Distribuição dos anúncios por tipo de quarto
- Análise da concentração territorial por meio de mapas e gráficos comparativos

<br> **Principais insights:** <br />
A oferta de anúncios apresenta forte concentração territorial. Distritos com menor número de bairros, como Manhattan (32 bairros), concentram grande volume de anúncios, atingindo cerca de 675 anúncios por bairro, valor aproximadamente 3 vezes superior à média geral de 220 anúncios por bairro. A análise de Pareto reforça esse padrão, mostrando que 35 dos 221 bairros são responsáveis por 80% da oferta total, evidenciando baixa dispersão territorial.
<br>
<br />

### Visão Receita (Preço)
Esta visão busca avaliar o posicionamento de preços e o potencial de geração de receita, com base em métricas estimadas.

<br> **Principais análises:** <br />
- Receita total estimada e receita média por anúncio
- Análise do ADR (preço médio diário) por distrito, tipo de quarto e categoria de preço
- Comparação entre volume de anúncios e potencial de receita
- Análises com e sem outliers para melhor entendimento da distribuição de preços

**Observação:**
As métricas de receita são utilizadas como referência para análises comparativas e de posicionamento, não representando valores reais transacionados.

<br> **Principais insights:** <br />
O potencial de receita estimada não depende exclusivamente do volume de anúncios. Embora Manhattan concentre tanto o maior número de anúncios quanto cerca de 65% da receita estimada, isso ocorre pela combinação de escala com posicionamento de preços mais elevado. Em contrapartida, Brooklyn adota uma estratégia de escala, com grande volume de anúncios, porém menor receita média por anúncio, enquanto Queens, mesmo com aproximadamente 1/4 do volume de anúncios de Brooklyn, apresenta maior eficiência por anúncio, refletindo um posicionamento de preço mais favorável.  
A análise com e sem outliers demonstra que valores atípicos exercem impacto relevante nas métricas de preço, reforçando a importância de separar comportamento padrão de casos extremos.
<br>
<br />

### Visão Engajamento (Avaliações)
A Visão de Engajamento analisa o comportamento dos hóspedes a partir das avaliações, tratadas como indicador indireto de engajamento.

<br> **Principais análises:** <br />
- Total de avaliações e média de avaliações por anúncio
- Percentual de anúncios com e sem avaliações
- Classificação dos anúncios em faixas de engajamento
- Distribuição do engajamento por distrito, tipo de quarto e categoria de preço
- Identificação de concentração de avaliações por bairro (Pareto)

<br> **Principais insights:** <br />
O engajamento dos hóspedes, medido pelo volume de avaliações, apresenta forte concentração territorial, com poucos bairros sendo responsáveis pela maior parte das interações registradas na plataforma. A análise de Pareto indica que aproximadamente 36 bairros (de 221) concentram cerca de 80% do total de avaliações, evidenciando a existência de hotspots de demanda e maior atratividade em regiões específicas da cidade.  
Embora não seja possível afirmar a concentração em nível de anúncio individual, os dados indicam que o engajamento está significativamente concentrado em determinadas áreas geográficas.
<br>
<br />

### Visão Anfitriões
Esta visão tem como foco compreender o perfil e o nível de profissionalização dos anfitriões.

<br> **Principais análises:** <br />
- Quantidade total de anfitriões
- Média de anúncios por anfitrião
- Classificação dos anfitriões por faixas de quantidade de anúncios
- Identificação de concentração de anúncios em poucos anfitriões
- Relação entre volume de anúncios e indicadores de engajamento

<br> **Principais insights:** <br />
A distribuição de anúncios por anfitrião é fortemente concentrada, com uma pequena parcela de anfitriões controlando uma parte significativa da oferta total, enquanto a maioria opera com poucos anúncios. Anfitriões com maior número de anúncios demonstram perfil mais profissionalizado, especialmente em distritos centrais, o que influencia tanto a escala da oferta quanto a dinâmica competitiva do mercado.
O crescimento da oferta está mais associado à expansão de portfólios de anfitriões já ativos do que à entrada proporcional de novos anfitriões, indicando uma tendência de consolidação da oferta ao longo do tempo.
<br>
<br />

## Conclusão
Do ponto de vista analítico, o projeto evidencia que a oferta de anúncios do Airbnb em Nova York é altamente concentrada, tanto em termos territoriais quanto no nível de bairros. Distritos com menor número de bairros, como Manhattan, apresentam uma densidade de anúncios significativamente superior à média geral, enquanto a análise de Pareto demonstra que uma parcela reduzida dos bairros concentra a maior parte da oferta disponível na cidade. As análises de engajamento e de receita estimada reforçam que volume de anúncios, isoladamente, não é  o único fator determinante de desempenho. Distritos com menor participação na oferta total, como Queens, podem apresentar desempenho relativo superior em métricas normalizadas, como receita média por anúncio, evidenciando a importância de análises comparativas para uma leitura mais precisa do mercado.
<br>
<br />
A introdução da dimensão de anfitriões adiciona uma camada estratégica relevante à análise, ao identificar padrões de concentração e profissionalização da oferta. Observa-se que uma parcela dos anfitriões concentra múltiplos anúncios, o que impacta diretamente a dinâmica de competição, a distribuição da oferta e o nível de profissionalização em determinadas regiões. 
<br>
<br />
De forma geral, os resultados mostram que o mercado de hospedagem em Nova York não é homogêneo nem previsível por uma única métrica. Oferta, receita, engajamento e perfil dos anfitriões variam de maneira significativa entre distritos e bairros, criando dinâmicas distintas dentro de um mesmo mercado.
Essas variações indicam que cada contexto exige uma estratégia específica: regiões com alta concentração de anúncios demandam abordagens diferentes daquelas com maior potencial de receita por anúncio, áreas com maior engajamento dos hóspedes apresentam comportamentos distintos de mercados mais pulverizados, e a presença de anfitriões profissionais influencia diretamente o nível de competitividade local.  

<br />
Assim, compreender essas nuances é fundamental para evitar decisões generalistas e para direcionar ações mais assertivas em frentes como precificação, posicionamento, expansão territorial e foco operacional.
<br>
<br />


## Entrega final do Projeto
**Acesso ao relatório:** https://app.powerbi.com/view?r=eyJrIjoiMjJjZjJhZWMtNTkzNS00MjYyLWEyMTItZjUxMDhiYmQwMTgzIiwidCI6IjMxYzdjNzA5LWZkOWQtNGIyNS05NTliLWI2ZGJiMGQ4Y2RlNiJ9
<br>
<br />

## Próximos Passos / Sugestões Futuras
Apesar de o projeto atender ao objetivo de fornecer uma visão geral sobre oferta, receita, engajamento e anfitriões, algumas extensões analíticas poderiam aprofundar ainda mais os insights:

### Análise de Concentração por Anúncio (Pareto por anúncio)
Atualmente, as análises de concentração são realizadas em nível territorial (bairros). 
Como evolução, recomenda-se a construção de uma análise de Pareto por anúncio (listing_id), permitindo avaliar se o volume de avaliações e de receita estimada também se concentra em poucos anúncios individuais dentro dos bairros mais relevantes.
<br>
<br />


### Avaliação de Risco por Outliers
Os outliers de preço foram tratados por meio do método IQR. Como evolução do projeto, recomenda-se analisar o perfil dos anúncios atípicos, avaliando sua concentração por distrito, tipo de quarto ou anfitrião, de forma a adicionar uma camada de análise de risco e exceção ao dashboard.
<br>
<br />


### Qualidade x Volume de Engajamento
O engajamento foi analisado com base no volume total de avaliações. Como evolução, recomenda-se diferenciar anúncios com avaliações recentes daqueles com avaliações predominantemente antigas, além de criar métricas de engajamento ativo que considerem apenas anúncios com atividade recente, permitindo distinguir relevância atual de histórico acumulado.
<br>
<br />

### Análise Avançada de Anfitriões
Na visão de anfitriões, análises adicionais podem explorar a relação entre volume de anúncios, engajamento e preço médio, bem como a identificação de perfis distintos de anfitriões, como ocasionais e profissionais.
<br>
<br />


### Modelos Preditivos e Segmentação
Como evolução analítica, o projeto pode incorporar técnicas de segmentação, como modelos de clusterização para agrupar anúncios ou bairros com base em preço, engajamento e tipo de quarto, além de modelos preditivos para estimar potencial de receita ou engajamento a partir das características dos anúncios.
