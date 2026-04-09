# Relatório – Avaliação de Similaridade de Perguntas com MPI

**Disciplina:** Computação Paralela e Distribuída
**Aluno(s):** Mateus Recalde da Fonseca Cotrim
**Professor:** Rafael Marconi Ramos
**Data:** 08/04/2026

---

# 1. Descrição do Problema

O programa desenvolvido tem como objetivo encontrar pares de perguntas similares dentro de um dataset do Quora (Quora Question Pairs). Para isso, ele compara todas as combinações possíveis de pares de perguntas usando similaridade cosseno com vetores TF-IDF.

O algoritmo basicamente percorre a metade superior de uma matriz de comparações (i, j) onde i < j, calculando a similaridade entre cada par. Com 5.000 perguntas, isso resulta em cerca de 12,5 milhões de comparações, o que torna o problema computacionalmente pesado e um bom candidato para paralelização.

- **Objetivo do programa:** encontrar os 20 pares de perguntas mais similares entre si dentro do conjunto de dados.
- **Volume de dados:** 5.000 perguntas → C(5000, 2) = 12.497.500 comparações.
- **Algoritmo utilizado:** similaridade cosseno sobre vetores TF-IDF. A paralelização divide as linhas `i` da matriz triangular entre os processos MPI.
- **Complexidade aproximada:** O(n²), onde n = 5.000.
- **Objetivo da paralelização:** dividir o trabalho das comparações entre múltiplos processos para reduzir o tempo total de execução.

---

# 2. Ambiente Experimental

Os testes foram realizados na seguinte máquina:

| Item                        | Descrição                                    |
| --------------------------- | -------------------------------------------- |
| Processador                 | Intel Core i5-12500 (12ª Geração)            |
| Número de núcleos           | 6 núcleos físicos / 12 processadores lógicos |
| Cache L1 / L2 / L3          | 480 KB / 7,5 MB / 18,0 MB                    |
| Memória RAM                 | 16,0 GB DDR4 4800 MT/s                       |
| Armazenamento               | SSD NVMe ADATA 512 GB                        |
| Sistema Operacional         | Windows 11                                   |
| Linguagem utilizada         | Python 3.x                                   |
| Biblioteca de paralelização | MPI (mpi4py)                                 |
| Compilador / Versão         | Python (interpretado) + mpiexec              |

---

# 3. Metodologia de Testes

O tempo de execução foi medido pelo próprio script `avaliador_mpi.py`, que imprime o tempo total ao final de cada execução no formato `Tempo total MPI: X.XX segundos`. Esse tempo é registrado pelo processo de rank 0, que é responsável por consolidar os resultados dos demais processos.

Para cada configuração foi realizada uma execução, utilizando sempre as mesmas 5.000 perguntas como entrada. Os testes foram feitos numa máquina de uso pessoal, sem nenhum isolamento especial de carga do sistema.

### Configurações testadas

- 1 processo (serial)
- 2 processos
- 4 processos
- 8 processos
- 12 processos

### Procedimento experimental

- **Execuções por configuração:** 1 (tempo único, sem cálculo de média)
- **Entrada:** 5.000 perguntas — 12.497.500 comparações no total
- **Ambiente:** Windows 11, máquina pessoal, com outros processos do sistema em execução
- **Medição:** tempo reportado diretamente pelo script ao final de cada execução

---

# 4. Resultados Experimentais

A tabela abaixo mostra os tempos de execução obtidos para cada configuração:

| Nº Processos MPI | Tempo de Execução (s) |
| ---------------- | --------------------- |
| 1                | 32,84                 |
| 2                | 24,43                 |
| 4                | 16,58                 |
| 8                | 12,21                 |
| 12               | 11,25                 |

---

# 5. Cálculo de Speedup e Eficiência

Para analisar o desempenho da paralelização, foram calculados o speedup e a eficiência de cada configuração usando as fórmulas abaixo.

### Speedup

```
Speedup(p) = T(1) / T(p)
```

Onde:

- **T(1)** = tempo da execução com 1 processo (serial)
- **T(p)** = tempo com p processos

### Eficiência

```
Eficiência(p) = Speedup(p) / p
```

Onde:

- **p** = número de processos MPI

---

# 6. Tabela de Resultados

Considerando T(1) = 32,84 s:

| Processos MPI | Tempo (s) | Speedup | Eficiência |
| ------------- | --------- | ------- | ---------- |
| 1             | 32,84     | 1,00    | 100,00%    |
| 2             | 24,43     | 1,34    | 67,10%     |
| 4             | 16,58     | 1,98    | 49,50%     |
| 8             | 12,21     | 2,69    | 33,60%     |
| 12            | 11,25     | 2,92    | 24,30%     |

---

# 7. Gráfico de Tempo de Execução

![Gráfico Tempo de Execução](graficos/grafico_tempo.png)

---

# 8. Gráfico de Speedup

![Gráfico Speedup](graficos/grafico_speedup.png)

---

# 9. Gráfico de Eficiência

![Gráfico Eficiência](graficos/grafico_eficiencia.png)

---

# 10. Análise dos Resultados

**O speedup obtido foi próximo do ideal?**

Não. Com 12 processos o speedup foi de apenas 2,92, bem longe do ideal que seria 12. Isso mostra que a aplicação não consegue aproveitar totalmente o paralelismo disponível, seja pelo desbalanceamento de carga ou pelo overhead de comunicação entre os processos.

**A aplicação apresentou escalabilidade?**

Parcialmente. Existe melhora de desempenho conforme o número de processos aumenta, mas os ganhos vão diminuindo. O maior salto acontece de 1 para 4 processos (tempo cai de 32,84 s para 16,58 s). Depois disso, de 8 para 12 processos, a diferença é bem pequena (12,21 s → 11,25 s), o que indica que a aplicação já está chegando num ponto de saturação.

**Em qual ponto a eficiência começou a cair?**

A eficiência já começa a cair a partir de 2 processos, indo de 100% para 67,10%. Conforme se aumenta o número de processos, ela cai cada vez mais, chegando a apenas 24,30% com 12 processos. Isso significa que boa parte dos recursos está sendo desperdiçada com overhead.

**O número de processos ultrapassa o número de núcleos físicos da máquina?**

Sim, a partir de 8 processos. O processador tem 6 núcleos físicos e 12 lógicos via Hyper-Threading. Então nas configurações de 8 e 12 processos, mais de um processo acaba compartilhando o mesmo núcleo físico. Com 12 processos, todos os processadores lógicos estão ocupados, o que explica o ganho tão pequeno em relação a 8 processos — o Hyper-Threading ajuda, mas não é a mesma coisa que ter núcleos físicos independentes, especialmente para cargas de processamento intenso como essa.

**Houve overhead de paralelização?**

Sim. Um problema bastante visível nos logs é o desbalanceamento de carga. A divisão do trabalho é feita por número de linhas `i`, mas linhas com índices menores têm mais comparações para fazer (porque `j` vai de i+1 até n-1). Com 2 processos, por exemplo, o Processo 0 faz 9.373.750 comparações enquanto o Processo 1 faz apenas 3.123.750. Como o tempo total depende do processo mais lento, grande parte do ganho teórico é perdida.

Outras causas identificadas para a perda de desempenho foram:

- **Desbalanceamento de carga:** divisão desigual das comparações entre os processos.
- **Overhead do MPI:** comunicação e coleta dos top-20 pares de cada processo no rank 0.
- **Carregamento do dataset:** cada processo carrega e processa o dataset de forma independente, o que é redundante.
- **Overhead do Python:** o interpretador em si é mais lento que linguagens compiladas, o que amplifica qualquer ineficiência.

---

# 11. Conclusão

No geral, a paralelização com MPI trouxe um ganho real de desempenho. O tempo caiu de 32,84 s para 11,25 s ao passar de 1 para 12 processos, uma redução de cerca de 66%. Porém, o speedup de 2,92× ficou muito abaixo do ideal teórico de 12×, o que mostra que a implementação atual tem bastante espaço para melhorias.

A configuração de 4 processos foi a que apresentou o melhor equilíbrio entre ganho de desempenho e eficiência, atingindo um speedup de ~2× com 49,50% de eficiência. A partir daí os retornos vão caindo bastante.

Para melhorar os resultados em trabalhos futuros, algumas mudanças que poderiam ser feitas são:

- Dividir a carga pelo número real de comparações e não pelo número de linhas, para balancear melhor o trabalho entre os processos.
- Fazer o broadcast dos vetores TF-IDF a partir do rank 0, evitando que cada processo recalcule tudo do zero.
- Usar NumPy de forma mais vetorizada para o cálculo de similaridade, reduzindo o overhead do Python.
- Explorar uma abordagem híbrida com MPI + OpenMP para aproveitar melhor os núcleos físicos disponíveis.
