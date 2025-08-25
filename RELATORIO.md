# Relatório do Laboratório 2 - Chamadas de Sistema

---

## Exercício 1a - Observação printf() vs 1b - write()

### Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: ___9__ syscalls
- ex1b_write: ___7__ syscalls

**2. Por que há diferença entre printf() e write()?**

```
Porque o printf() é uma função de uma bliblioteca que manda o texto formatado para o buffer e depis chama o write(), já ´white() é uma função do systema(syscall) que Vai direto para o kernel pedir que escreva os dados que te passei e não passa por um buffer.
```

**3. Qual implementação você acha que é mais eficiente? Por quê?**

```
Acho que anbos podem ser uteis dependendo da situação mas eu faria preferencia para o printf() pois ele permite formatar os dados e usa o buffer que só faz o write quando está cheio ou enconta \n, o que é mais eficiente quando se tem que fazer varias chamadas diferentes.
```

---

## Exercício 2 - Leitura de Arquivo

### Resultados da execução:
- File descriptor: __3___
- Bytes lidos: __127___

### Comando strace:
```bash
strace -e open,read,close ./ex2_leitura
```

### Análise

**1. Por que o file descriptor não foi 0, 1 ou 2?**

```
Porque os descritores 0, 1 e 2 já são reservados pelo sistema, ou seja o kernel automaticamente abre 3 descritores padrão, um para a entrada do teclado que é p 0 ou 'stdin', outro para a saida de informações para a tela o 1 ou stdout, e por ultimo outra saida para a tela mas esta agora focada em erros. Sendo assim a primeiro por ta livre era a 3 por isso não foi a o, 1 ou 2.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
Sabemos que o arquivo foi lido completamente, pela linha "read(3, "Esta e a primeira linha do arqui"..., 127) = 127" pois ela nos fala que retornou exatamente 127 bytes, e em seguida o programa mostrou todo o conteúdo esperado do arquivo (todas as linhas).
```

---

## Exercício 3 - Contador com Loop

### Resultados (BUFFER_SIZE = 64):
- Linhas: ___24__ (esperado: 25)
- Caracteres: __1299___
- Chamadas read(): __21___
- Tempo: __0,000123___ segundos

### Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |        82         |     0.000180      |
| 64          |        72         |     0.000123      |
| 256         |        6          |     0.000071      |
| 1024        |        2          |     0.000068      |

### Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Apesar de ser não haver uma diferença visual no númeroa de chamadas podemos ver que a velocidade tende a ser mais rapida se o buffer for maior portanto acredito que Buffers maiores reduzem o número de syscalls em arquivos grandes e tornam cada chamada mais eficiente, apessar de não ser muito visivel nesse caso.
```

**2. Como você detecta o fim do arquivo?**

```
O fim do arquivo é detectado quando "(bytes_lidos = read(fd, buffer, BUFFER_SIZE)) > 0" da falso, ou seja não foi possivel ler mais nenhum dado do arquivo logo ele acabou.
```

---

## Exercício 4 - Cópia de Arquivo

### Resultados:
- Bytes copiados: __1364___
- Operações: __6___
- Tempo: __0.000453___ segundos
- Throughput: __2940.47___ KB/s

### Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [x] Idênticos [ ] Diferentes

### Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Para garantir que não deixamos de escrever algo no arquivo novo devido a alguma interrupção do sistema ou tramanho de buffer, ou seja se n~qao fizermos essa verigficação podemos acabar perdendo dados
```

**2. Que flags são essenciais no open() do destino?**

```
As 3 flags são importantes(O_WRONLY | O_CREAT | O_TRUNC), o O_WRONLY abre o arquivo apenas para escrita necessario pois estamos usando write(), já o O_CREAT cria o arquivo se ele não existir, sem ele teriammos um erro no open() se o arquivo não existisse, e por ultimo O_TRUNC se o arquivo já existir, zera o conteúdo antes de escrever, o que é necessario se não quisermos que o conteudo antigo fique no arquivo.
```

---

## Análise Geral

### Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
Uma syscall é o mecanismo pelo qual um programa em modo usuário pede ao kernel que execute operações privilegiadas (como acessar disco ou memória), ocorrendo uma transição temporária para modo kernel e depois o retorno ao usuário com o resultado.
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
File descriptors são essenciais porque fornecem uma forma padronizada de o programa acessar qualquer recurso de I/O através do kernel.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Pelo que pude observar o tamanhça do buffer interfere na peformance de forma que quanto menor o buffer o processo fica mais lento, porém pelo que pude ver um maior também nem sempre é a solução pois ele pode estar diperdiçando memoria, então o ideal é analizar o tamanho da informação que precisamos e definir um tamanho de buffer adequado.
```

### Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** ___o programa__

**Por que você acha que foi mais rápido?**

```
Porque o programa fazia o contato mais rapido com o kernel pela syscall
```

---

## Entrega

Certifique-se de ter:
- [x] Todos os códigos com TODOs completados
- [x] Traces salvos em `traces/`
- [x] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```