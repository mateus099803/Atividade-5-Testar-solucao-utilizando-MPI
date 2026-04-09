# Relatório de Avaliação de Similaridade de Perguntas com MPI

**Disciplina:** Computação Paralela e Distribuída
**Aluno(s):** _(preencher)_
**Turma:** _(preencher)_
**Professor:** _(preencher)_
**Data:** 08/04/2026

---

# 1. Descrição do Problema

O programa implementa um avaliador de similaridade entre pares de perguntas utilizando o dataset Quora Question Pairs. O algoritmo percorre todas as combinações possíveis de pares de perguntas (i, j) com i < j — ou seja, a metade superior da matriz de similaridade — e calcula a similaridade cosseno entre os vetores TF-IDF de cada par.

**Qual é o objetivo do programa?**
Identificar os pares de perguntas semanticamente mais similares dentro de um conjunto de 5.000 perguntas extraídas do dataset, retornando um ranking com os 20 pares de maior similaridade.

**Qual o volume de dados processado?**
5.000 perguntas, resultando em 12.497.500 comparações par a par (combinações C(5000, 2)).

**Qual algoritmo foi utilizado?**
Similaridade cosseno sobre representações TF-IDF das perguntas. A paralelização se dá pela divisão das linhas `i` da matriz triangular entre os processos MPI, onde cada processo fica responsável por um intervalo contíguo de índices `i` e compara cada linha com todas as colunas `j > i`.

**Qual a complexidade aproximada do algoritmo?**
O(n²) no número de comparações, onde n = 5.000. O cálculo de similaridade cosseno é O(d) por par, onde d é a dimensão dos vetores TF-IDF.

**Qual o objetivo da paralelização?**
Distribuir a carga das 12,5 milhões de comparações entre múltiplos processos MPI para reduzir o tempo total de execução, aproveitando os núcleos físicos disponíveis no processador.

---

# 2. Ambiente Experimental

| Item                        | Descrição                                              |
| --------------------------- | ------------------------------------------------------ |
| Processador                 | Intel Core i5-12500 (12ª Geração), 3,00 GHz base       |
| Número de núcleos           | 6 núcleos físicos / 12 processadores lógicos (Hyper-Threading) |
| Cache L1 / L2 / L3          | 480 KB / 7,5 MB / 18,0 MB                             |
| Memória RAM                 | 16,0 GB DDR4 4800 MT/s (1 módulo DIMM, 1 de 2 slots)  |
| Armazenamento               | SSD NVMe ADATA 512 GB (SM2P41C3Q), tempo de resposta 1,6 ms |
| Sistema Operacional         | Windows 11                                             |
| Linguagem utilizada         | Python 3.x                                             |
| Biblioteca de paralelização | MPI (mpi4py)                                           |
| Compilador / Versão         | Python (interpretado) + mpiexec                        |

---

# 3. Metodologia de Testes

O tempo de execução foi medido internamente pelo próprio script `avaliador_mpi.py`, que registra o tempo total do processo principal (Processo 0) desde o início da computação até a consolidação dos resultados via `MPI_Gather` ou equivalente.

Os experimentos foram executados com uma única rodada por configuração, utilizando os tempos reportados diretamente pelo programa. O tamanho da entrada foi fixado em **5.000 perguntas** para todas as execuções.

### Configurações testadas

Os experimentos foram realizados nas seguintes configurações de processos MPI:

- 1 processo (versão serial)
- 2 processos
- 4 processos
- 8 processos
- 12 processos

### Procedimento experimental

- **Número de execuções:** 1 execução por configuração (tempo único, não média)
- **Tamanho da entrada:** 5.000 perguntas — 12.497.500 comparações
- **Condições de execução:** Windows 11, máquina pessoal, sem isolamento de carga
- **Medição:** Tempo total reportado pelo script ao final da execução (`Tempo total MPI: X.XX segundos`)

---

# 4. Resultados Experimentais

| Nº Processos MPI | Tempo de Execução (s) |
| ---------------- | --------------------- |
| 1                | 32,84                 |
| 2                | 24,43                 |
| 4                | 16,58                 |
| 8                | 12,21                 |
| 12               | 11,25                 |

---

# 5. Cálculo de Speedup e Eficiência

### Speedup

```
Speedup(p) = T(1) / T(p)
```

Onde:

- **T(1)** = tempo da execução serial (1 processo)
- **T(p)** = tempo com p processos

### Eficiência

```
Eficiência(p) = Speedup(p) / p
```

Onde:

- **p** = número de processos MPI

---

# 6. Tabela de Resultados

T(1) = 32,84 s

| Processos MPI | Tempo (s) | Speedup          | Eficiência        |
| ------------- | --------- | ---------------- | ----------------- |
| 1             | 32,84     | 1,00             | 100,00%           |
| 2             | 24,43     | 1,34             | 67,10%            |
| 4             | 16,58     | 1,98             | 49,50%            |
| 8             | 12,21     | 2,69             | 33,60%            |
| 12            | 11,25     | 2,92             | 24,30%            |

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
Não. O speedup máximo obtido com 12 processos foi de apenas 2,92, enquanto o ideal seria 12. Isso indica que a maior parte do trabalho não foi paralelizável de forma eficiente, ou que o overhead de comunicação e a divisão desigual de carga limitaram os ganhos.

**A aplicação apresentou escalabilidade?**
Parcialmente. Há ganho de desempenho com o aumento de processos, mas os retornos decrescem rapidamente. O salto de 1 para 4 processos é o mais expressivo (tempo cai de 32,84 s para 16,58 s). De 8 para 12 processos, o ganho é marginal (12,21 s → 11,25 s), evidenciando saturação.

**Em qual ponto a eficiência começou a cair?**
Já a partir de 2 processos a eficiência caiu para 0,67 (abaixo do ideal 1,0), com queda acentuada a cada configuração. A partir de 8 processos, a eficiência já está abaixo de 0,34, tornando o acréscimo de processos pouco vantajoso.

**O número de threads ultrapassa o número de núcleos físicos da máquina?**
Sim, a partir de 8 processos. A máquina possui **6 núcleos físicos** e **12 processadores lógicos** via Hyper-Threading. As configurações de 8 e 12 processos já ultrapassam o número de núcleos físicos, compartilhando núcleos entre processos lógicos. A configuração de 12 processos ocupa todos os processadores lógicos disponíveis, o que explica o ganho marginal entre 8 e 12 processos — os núcleos físicos já estavam saturados e o Hyper-Threading não oferece o mesmo desempenho que núcleos dedicados para cargas computacionais intensas.

**Houve overhead de paralelização?**
Sim. A distribuição das linhas `i` entre os processos é feita de forma que o Processo 0 recebe a maior fatia (linhas iniciais, que possuem mais comparações por linha), enquanto os processos finais recebem fatias menores. Isso gera **desbalanceamento de carga**: por exemplo, com 2 processos, o Processo 0 realiza 9.373.750 comparações enquanto o Processo 1 realiza apenas 3.123.750. O tempo total é determinado pelo processo mais lento (Processo 0), reduzindo o ganho efetivo.

**Causas identificadas para perda de desempenho:**
- **Desbalanceamento de carga:** a divisão linear dos índices `i` não distribui o trabalho igualmente, pois linhas com índices menores possuem mais comparações (`j` de i+1 até n-1).
- **Overhead de comunicação MPI:** coleta e merge dos top-20 pares de cada processo no rank 0.
- **Carregamento e vetorização do dataset:** cada processo carrega e processa o dataset de forma independente (sem paralelismo nessa etapa), o que pode ser um gargalo inicial.
- **GIL e overhead do Python:** o interpretador Python introduz overhead em comparação a implementações em C/C++.

---

# 11. Conclusão

O experimento demonstrou que a paralelização via MPI trouxe ganho real de desempenho — o tempo caiu de 32,84 s para 11,25 s ao passar de 1 para 12 processos, uma redução de aproximadamente 66%. No entanto, o speedup obtido (2,92×) ficou muito abaixo do ideal teórico (12×), evidenciando que a aplicação não escala linearmente.

O **melhor custo-benefício** foi observado na configuração de 4 processos, onde o speedup de ~2× foi atingido com eficiência de 49% — um equilíbrio razoável entre ganho de desempenho e utilização dos recursos.

As principais limitações são o **desbalanceamento de carga** na divisão dos índices triangulares e os overheads inerentes ao Python e à comunicação MPI. Melhorias possíveis incluem:

- Distribuição balanceada da carga considerando o número real de comparações por processo (divisão por número de operações, não por número de linhas);
- Pré-computação e broadcast dos vetores TF-IDF apenas uma vez, no rank 0;
- Migração para implementação em C com mpi4py apenas para orquestração, ou uso de NumPy vetorizado para o cálculo de similaridade;
- Avaliação de abordagens híbridas MPI + OpenMP para aproveitar melhor os núcleos físicos disponíveis.
