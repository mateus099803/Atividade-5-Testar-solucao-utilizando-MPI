# Relatório – Avaliação de Similaridade de Perguntas com MPI

**Disciplina:** Computação Paralela e Distribuída
**Aluno(s):** Mateus Recalde da Fonseca Cotrim
**Professor:** Rafael Marconi Ramos
**Data:** 08/04/2026

---

# 1. Descrição do Problema
 
O programa compara pares de perguntas de um dataset do Quora para encontrar as mais parecidas entre si. Ele usa similaridade cosseno com TF-IDF para medir o quanto duas perguntas se parecem.
 
Com 5.000 perguntas, são feitas cerca de 12,5 milhões de comparações (todas as combinações possíveis de pares). Por ser um volume grande de processamento, o problema foi paralelizado com MPI para dividir esse trabalho entre múltiplos processos.
 
- **Objetivo:** encontrar os 20 pares de perguntas mais similares no dataset.
- **Entrada:** 5.000 perguntas → 12.497.500 comparações.
- **Algoritmo:** similaridade cosseno sobre vetores TF-IDF.
- **Complexidade:** O(n²), com n = 5.000.
- **Por que paralelizar:** reduzir o tempo de execução distribuindo as comparações entre processos.
 
---
 
# 2. Ambiente Experimental
 
| Item                        | Descrição                                                      |
| --------------------------- | -------------------------------------------------------------- |
| Processador                 | Intel Core i5-12500 (12ª Geração), 3,00 GHz base              |
| Número de núcleos           | 6 núcleos físicos / 12 lógicos (Hyper-Threading)              |
| Cache L1 / L2 / L3          | 480 KB / 7,5 MB / 18,0 MB                                     |
| Memória RAM                 | 16,0 GB DDR4 4800 MT/s                                        |
| Armazenamento               | SSD NVMe ADATA 512 GB, tempo de resposta 1,6 ms               |
| Sistema Operacional         | Windows 11                                                     |
| Linguagem                   | Python 3.x                                                     |
| Biblioteca MPI              | mpi4py                                                         |
| Execução                    | mpiexec                                                        |
 
---
 
# 3. Metodologia de Testes
 
O tempo foi medido pelo próprio script, que registra quanto tempo o processo principal levou desde o início até reunir os resultados de todos os outros processos.
 
Foram testadas 5 configurações diferentes, sempre com as mesmas 5.000 perguntas. Cada configuração foi executada uma vez, sem repetições para cálculo de média. Os testes foram feitos numa máquina de uso pessoal com o sistema operacional rodando normalmente.
 
### Configurações testadas
 
- 1 processo (execução serial)
- 2 processos
- 4 processos
- 8 processos
- 12 processos
 
---
 
# 4. Resultados Experimentais
 
| Nº Processos | Tempo de Execução (s) |
| ------------ | --------------------- |
| 1            | 32,84                 |
| 2            | 24,43                 |
| 4            | 16,58                 |
| 8            | 12,21                 |
| 12           | 11,25                 |
 
---
 
# 5. Cálculo de Speedup e Eficiência
 
### Speedup
 
O speedup mostra quantas vezes a versão paralela foi mais rápida que a serial.
 
```
Speedup(p) = T(1) / T(p)
```
 
- **T(1)** = tempo com 1 processo
- **T(p)** = tempo com p processos
 
### Eficiência
 
A eficiência mostra o quanto cada processo está sendo aproveitado de verdade.
 
```
Eficiência(p) = Speedup(p) / p
```
 
- **p** = número de processos
 
---
 
# 6. Tabela de Resultados
 
T(1) = 32,84 s
 
| Processos | Tempo (s) | Speedup | Eficiência |
| --------- | --------- | ------- | ---------- |
| 1         | 32,84     | 1,00    | 100,00%    |
| 2         | 24,43     | 1,34    | 67,10%     |
| 4         | 16,58     | 1,98    | 49,50%     |
| 8         | 12,21     | 2,69    | 33,60%     |
| 12        | 11,25     | 2,92    | 24,30%     |
 
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
 
**O speedup foi próximo do ideal?**
 
Não. Com 12 processos o speedup foi 2,92, quando o ideal seria 12. O programa ficou longe de aproveitar todo o potencial do paralelismo.
 
**A aplicação escalou bem?**
 
Mais ou menos. O tempo caiu bastante de 1 para 4 processos, mas depois disso os ganhos foram bem menores. De 8 para 12 processos a diferença foi quase nula (12,21 s para 11,25 s).
 
**Em que ponto a eficiência caiu?**
 
Já caiu com 2 processos (de 100% para 67%). Com 12 processos chegou a 24%, ou seja, mais de 75% da capacidade paralela estava sendo desperdiçada.
 
**O número de processos passou do número de núcleos físicos?**
 
Sim, a partir de 8 processos. A máquina tem 6 núcleos físicos e 12 lógicos (Hyper-Threading). Com 8 e 12 processos, mais de um processo passa a dividir o mesmo núcleo físico, o que reduz o desempenho. Por isso o ganho entre 8 e 12 processos foi tão pequeno.
 
**Teve overhead de paralelização?**
 
Sim. O principal problema foi o desbalanceamento de carga: o programa divide as linhas `i` igualmente entre os processos, mas as primeiras linhas têm muito mais comparações para fazer que as últimas. Com 2 processos, o Processo 0 fez 9,3 milhões de comparações enquanto o Processo 1 fez só 3,1 milhões. O tempo total é o do processo mais lento, então muito tempo ficou sendo desperdiçado esperando o Processo 0 terminar.
 
Outros fatores que contribuíram:
 
- Comunicação entre processos para juntar os resultados no final.
- Cada processo carrega e processa o dataset do zero, sem compartilhar nada.
- O Python em si já é mais lento que linguagens compiladas.
 
---
 
# 11. Conclusão
 
A paralelização funcionou e trouxe ganho de desempenho: o tempo caiu de 32,84 s para 11,25 s com 12 processos, uma redução de 66%. Porém, o speedup de 2,92× ficou muito abaixo do ideal (12×), mostrando que o programa não aproveita bem o paralelismo disponível.
 
A melhor relação custo-benefício foi com 4 processos, que teve um speedup próximo de 2× com 49% de eficiência. Aumentar além disso trouxe pouco retorno.
 
Para melhorar, seria interessante:
 
- Dividir o trabalho pelo número real de comparações, não pelo número de linhas.
- Compartilhar os vetores TF-IDF entre os processos em vez de cada um calcular do zero.
- Usar mais NumPy para aproveitar melhor o processamento em Python.
- Testar uma abordagem híbrida com MPI + threads para usar melhor os núcleos disponíveis.
 
