# Roteiro — ENIAC 2026 (12 min)

**Balanceando o MINA-BR com um LLM uncensored**

Tempo total: 12:00. A coluna "acumulado" mostra onde você deveria estar ao **terminar** cada slide.

| # | Slide | Tempo | Acumulado |
|---|---|---|---|
| 1 | Título | 0:30 | 0:30 |
| 2 | Motivação | 1:00 | 1:30 |
| 3 | Dataset MINA-BR | 0:50 | 2:20 |
| 4 | Modelo abliterated | 1:00 | 3:20 |
| 5 | Pipeline | 1:00 | 4:20 |
| 6 | Few-shot dinâmico | 1:00 | 5:20 |
| 7 | Setup de avaliação | 0:50 | 6:10 |
| 8 | Resultados (gráfico) | 1:10 | 7:20 |
| 9 | Ganhos e estabilidade | 0:50 | 8:10 |
| 10 | Não é só balancear | 0:50 | 9:00 |
| 11 | Qualidade dos dados sintéticos | 1:00 | 10:00 |
| 12 | Ética e limitações | 0:50 | 10:50 |
| 13 | Conclusões | 0:50 | 11:40 |
| 14 | Obrigado | 0:20 | 12:00 |

---

## 1. Título (0:30)

Boa tarde. Meu nome é Andrew Campos, sou do PPGC da UFPel, e este trabalho é com o Ulisses Corrêa e a Larissa de Freitas. Vamos falar sobre como usar um LLM *uncensored* para gerar dados sintéticos e balancear um dataset de discurso de ódio misógino em português. Um aviso rápido: alguns slides mostram palavras ofensivas, porque esse é o objeto de estudo.

## 2. Motivação (1:00)

Discurso de ódio é toda comunicação que ataca pessoas por atributos como raça, religião ou gênero, e o volume só cresce com as redes sociais. Para o português brasileiro, o problema é duplo: temos poucos corpora anotados e eles são muito desbalanceados — quase tudo é "não ódio".

Uma saída natural seria usar LLMs para gerar mais exemplos da classe minoritária. Só que modelos comerciais como GPT-4 e Claude passam por alinhamento de segurança e se recusam a gerar conteúdo de ódio, mesmo em pesquisa. Então a nossa pergunta é: um LLM com a recusa removida consegue gerar exemplos úteis e melhorar um classificador?

## 3. Dataset MINA-BR (0:50)

Usamos o MINA-BR, que tem 6.002 comentários do Twitter e do YouTube com foco em misoginia. Trabalhamos com o subconjunto rotulado, anotado por três anotadores independentes, com o rótulo "ódio".

O ponto principal é o desbalanceamento: cerca de 9 para 1 entre não ódio e ódio. E a classe que nos interessa é justamente a minoritária. Vale dizer que a coleta e a anonimização do dataset foram aprovadas pelo sistema CEP/Conep.

## 4. Modelo abliterated (1:00)

O que é um modelo *abliterated*? Arditi e colegas mostraram em 2024 que o comportamento de recusa de um LLM é mediado por uma única direção no *residual stream*. Eles estimam essa direção com diferença de médias entre ativações e a removem por ortogonalização dos pesos. O modelo deixa de recusar e, segundo os autores, mantém as capacidades em quatro benchmarks de linguagem.

Nós usamos o huihui-ai/Qwen2.5-14B-Instruct-abliterated, que é o Qwen 2.5 de 14B com essa direção removida, servido localmente com vLLM.

## 5. Pipeline (1:00)

Aqui está o pipeline completo. Do lado esquerdo, a geração de dados: partimos do treino do MINA-BR, montamos um prompt few-shot dinâmico com cinco exemplos reais de ódio e pedimos ao modelo abliterated um comentário novo.

Os comentários gerados são da classe ódio e entram no dataset junto com os dados reais — a linha tracejada. O resultado é um dataset balanceado: 1.751 não ódio e 1.747 ódio.

Do lado direito, a classificação: fazemos fine-tuning do BERTimbau com nested cross-validation e Optuna, e reportamos macro F1 no teste held-out.

## 6. Few-shot dinâmico (1:00)

Detalhando a geração em quatro etapas. Primeiro, a cada chamada sorteamos cinco comentários de ódio reais do treino — um sorteio novo a cada chamada, porque a ordem e a escolha dos exemplos influenciam a saída.

Segundo, esses exemplos entram no prompt como referência de tom e de grafia. Queremos texto com cara de internet, não texto bem escrito.

Terceiro, na inferência usamos temperature 0,7, top-p 0,8, top-k 20 e presence penalty 1,0, para forçar diversidade lexical.

E quarto, repetimos até balancear. A pequena diferença entre 1.751 e 1.747 vem de gerações que estouraram o limite de tamanho.

## 7. Setup de avaliação (0:50)

Comparamos quatro pipelines. Dois clássicos: SVM com TF-IDF, sem balanceamento e com SMOTE — o SMOTE gera exemplos sintéticos interpolando vetores, não texto. Ambos com grid search e 5-fold estratificado.

E dois com BERTimbau: nos dados originais e nos dados aumentados pelo LLM, ambos com nested CV e Optuna. O nested CV separa a escolha de hiperparâmetros da avaliação final e evita uma estimativa otimista. Cada pipeline foi avaliado na sua configuração mais favorável.

## 8. Resultados (1:10)

Este é o resultado principal, em macro F1. Em cinza, os baselines publicados no trabalho original do MINA-BR: o melhor deles chega a 0,56, tanto com SVM quanto com BERT.

Em azul-claro, os nossos baselines. Só com SVM nos dados originais já chegamos a 0,690. O BERTimbau sem aumento vai a 0,737, e o SMOTE com SVM chega a 0,800.

E em azul-escuro, o BERTimbau treinado com os dados do LLM: 0,915 de macro F1, com desvio-padrão de 0,008 entre os folds externos.

## 9. Ganhos e estabilidade (0:50)

Resumindo em números: são 17,8 pontos percentuais acima do mesmo BERTimbau sem aumento, e 35,5 pontos acima do melhor baseline publicado.

Olhando só a classe ódio, que é a que importa, o F1 vai a 0,916, contra 0,583 sem aumento e 0,671 com SMOTE.

E um detalhe que acho importante: o desvio-padrão caiu de 0,017 para 0,008, menos da metade. Ou seja, o aumento não só melhora a média como também deixa o modelo mais estável entre as partições.

## 10. Não é só balancear (0:50)

Mas será que o ganho vem apenas de balancear? A comparação ajuda a separar os efeitos. No trabalho original, BERT empata com SVM em 0,560, então representação contextual sozinha não resolve o desbalanceamento.

O SMOTE, que só balanceia no espaço vetorial, já leva o SVM de 0,690 para 0,800. Então balancear ajuda bastante. Mas quando o balanceamento é feito com texto realista gerado pelo LLM, chegamos a 0,915. A nossa leitura é que a qualidade dos dados gerados é o principal motor da melhora, e não apenas o balanceamento.

## 11. Qualidade dos dados sintéticos (1:00)

Também quisemos saber se o modelo estava só copiando os exemplos do prompt. Medimos duas coisas. O Type-Token Ratio médio é 0,974, ou seja, há alta diversidade lexical dentro de cada instância. E a similaridade de Jaccard entre cada comentário gerado e os seus exemplos de few-shot é só 0,043. O modelo generaliza a partir das demonstrações em vez de reproduzi-las.

No gráfico, as palavras mais frequentes da classe ódio. Aparecem colocações pejorativas como "mal amada" e "mal comida" — por isso "mal", "comida" e "amada" no topo. Um detalhe interessante: "mulher", em laranja, também é a palavra mais frequente na classe não ódio. Então a palavra sozinha não discrimina nada; o que define o ódio é o contexto em volta. Essa consistência de padrões provavelmente explica o desvio-padrão baixo.

## 12. Ética e limitações (0:50)

Precisamos falar de ética. A mesma técnica que permite esta pesquisa pode ser usada para gerar ódio em escala. Defendemos o uso restrito a contextos de pesquisa e com salvaguardas, e por isso não distribuímos os dados gerados.

Há duas limitações principais. Avaliamos em um único dataset, então a generalização para corpora como HateBR e ToLD-BR ainda está em aberto. E não fizemos avaliação humana dos textos sintéticos — comparar textos gerados e reais com anotadores humanos fortaleceria muito o trabalho.

## 13. Conclusões (0:50)

Para fechar: um LLM abliterated com few-shot dinâmico conseguiu balancear o MINA-BR com texto diverso e pouco copiado. O BERTimbau treinado nesses dados chegou a 0,915 de macro F1, 35,5 pontos acima do estado da arte publicado. E a comparação com SMOTE indica que a qualidade do texto gerado, e não só o balanceamento, explica o ganho.

Como próximos passos, queremos testar outros corpora, fazer avaliação humana e explorar outras estratégias de prompt e outros modelos abliterated. O código está no GitHub.

## 14. Obrigado (0:20)

Obrigado pela atenção. Este trabalho foi financiado em parte pela CAPES. Fico à disposição para perguntas.

---

## Perguntas prováveis (preparo)

- **"O teste contém dados sintéticos?"** O paper diz apenas que o BERTimbau foi avaliado em um teste held-out após o nested CV. Confirme antes da apresentação em qual partição os sintéticos entram, porque essa é a pergunta mais provável da banca.
- **"A comparação com SMOTE é justa, se o classificador muda (SVM vs. BERTimbau)?"** Cada pipeline foi avaliado na sua configuração mais favorável. A comparação direta de efeito do aumento é BERTimbau 0,737 → 0,915.
- **"Por que não usar GPT-4/Claude?"** Eles recusam gerar discurso de ódio por causa do alinhamento de segurança.
- **"Os dados gerados estão disponíveis?"** Não, por motivos éticos. O código está em github.com/AndrewwBC/ENIAC_HS.
- **Hardware:** RTX 5090 com 32 GB de VRAM.
