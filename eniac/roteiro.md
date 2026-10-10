# Slide 2
No contexto de treinamento de modelos de IA, dados balanceados favorecem uma
performance mais adequada tendo em vista que o modelo será exposto a uma gama equilibrada
de amostras de ambas as classes de um dataset, como ódio e não ódio.
Nesse cenário, a anotação manual de dados se mostra trabalhosa e demorada.
Uma alternativa é o uso de LLMs para a tarefa.

O principal impeditivo é o fato de que os modelos são alinhados para que tenham
um comportamento adequado diante dos usuários. Sendo assim, tais modelos
se recusam a gerar dados que simulem discurso de ódio.

Para mitigar este problema, exploramos LLMs sem censura para a geração dos dados.

# Slide 3
O dataset original MINA-BR é desbalanceado. A razão entre as classes é de 
nove pra um, 9:1, fazendo com que o modelo tenha exposição a poucos dados 
rotulados como ódio.
