# Slide 1
Boa tarde. Meu nome é Andrew Campos, sou do PPGC da UFPel, e este trabalho foi feito em conjunto com o Ulisses Corrêa e a Larissa de Freitas.

Vamos mostrar como um LLM sem censura pode gerar dados sintéticos para balancear um dataset de discurso de ódio misógino em português brasileiro.

Um aviso: alguns slides contêm linguagem ofensiva, pois esse é o objeto de estudo.

# Slide 2
No contexto de treinamento de modelos de IA, dados balanceados favorecem uma
performance mais adequada tendo em vista que o modelo será exposto a uma gama equilibrada
de amostras de ambas as classes de um dataset, como ódio e não ódio.
Nesse cenário, a anotação manual de dados se mostra trabalhosa e demorada.
Uma alternativa é o uso de LLMs para a tarefa.

O principal impeditivo é o fato de que os modelos são alinhados para que tenham
um comportamento adequado diante dos usuários. Sendo assim, tais modelos
se recusam a gerar dados que simulem discurso de ódio.

Para mitigar este problema, exploramos LLMs sem censura para aumentar o dataset MINA-BR.

# Slide 3
O dataset original MINA-BR é desbalanceado. A razão entre as classes é de 
nove pra um, 9:1, fazendo com que o modelo tenha exposição a poucos dados 
rotulados como ódio. Parte deste dataset serviu como exemplos para o modelo sem censura.

# Slide 4
O modelo sem censura ou abliterado passa por um processo
de abliteração que retira a componente de censura das camadas internas. Apesar do processo alterar o modelo, a literatura mostra que a performance do modelo se mantém praticamente igual ao original em alguns benchmarks.

# Slide 5
Nós empregamos o modelo apenas nas amostras rotuladas como ódio. Sendo assim, geramos 5 datasets balanceados através de amostragens diferentes entre 5 seeds.

Cada dataset foi utilizado para treinar um modelo BERTimbau com uma head de classificação.

# Slide 6 
De maneira mais detalhada. Cada balanceamento contou com a aplicação da técnica de few-shot, que consiste em dar exemplos ao LLM. Cada dataset foi gerado com um único conjunto de exemplos. Ou seja, tivemos 5 conjuntos diferentes, um por seed.

# Slide 7 
Os dados gerados foram utilizados em algoritmos como SVM e SMOTE para que pudéssemos formalizar um baseline no contexto dos novos dados.

Ao BERTimbau coube o uso de Nested Cross Validation para encontrarmos os melhores hiper-parâmetros utilizando Optuna.

# Slide 8
A nossa abordagem, desde o SVM com TF-IDF, até o BERTimbau, superam os baselines originais. Conseguimos um macro F1 de 0,915. Este valor variou pouco dado o desvio padrão inexpressivo nas cinco seeds.

# Slide 9
Em números, o ganho foi de 17,8 pontos percentuais sobre o mesmo BERTimbau sem aumento, e de 35,5 pontos sobre o melhor baseline publicado.

Na classe ódio, que é a que nos interessa, o F1 chegou a 0,916, contra 0,583 sem aumento e 0,671 com SMOTE.

Além disso, o desvio padrão caiu de 0,017 para 0,008, menos da metade. Ou seja, o aumento não só melhora a média, como também deixa o modelo mais estável entre as partições.

# Slide 10
O balanceamento só terá relevância se os dados sintéticos refletirem
parte da estrutura semântica e sintática dos dados reais. O treino em dados reais e teste nos dados sintéticos evidenciam que os dados gerados conseguiram suprir este requisito.

# Slide 11 
Para além das métricas dos modelos, analisamos os dados utilizando Type-Token Ratio e Jaccard. 
O TTR demonstra uma alta diversidade lexical, enquanto que a métrica Jaccard afirma que há sobreposição quase nula com os exemplos, ou seja, o modelo não copiou as informações, apenas se baseou nelas.

Apresentamos no gráfico o top-10 das palavras nas classes de ódio, aumentadas. O gráfico demonstra claramente alta ocorrência de palavras como "mal", "mulher", "sabe" e "sempre". Palavras que aparecem em frases como "mulher mal comida", "mulher mal amada".

# Slide 12

Um aspecto que devemos salientar neste trabalho é que, embora os modelos sem censura estejam disponíveis, defendemos o uso legal e ético destes modelos, tendo em vista que podem ser usados para geração de discurso de ódio de maneira automatizada.

Nossas limitações destacam o uso de apenas um dataset para aumento de dados e a ausência de avaliação humana dos dados gerados, utilizando apenas a performance dos modelos como interpretação qualitativa.

# Slide 13

Concluindo, demonstramos que o modelo sem censura viabilizou o balanceamento do dataset MINA-BR. Superamos os baselines com esta estratégia. Demonstramos que o ganho não vem apenas do volume, mas também da qualidade dos dados mediante métricas como TTR e Jaccard.

Nossos próximos passos: outros datasets, avaliação humana, diferentes estratégias de prompt e a exploração de outros modelos abliterados.

# Slide 14

Obrigado!