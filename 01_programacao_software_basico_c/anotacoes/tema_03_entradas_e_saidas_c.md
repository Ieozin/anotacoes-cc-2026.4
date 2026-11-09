# Tema 03: Entradas e Saídas em Linguagem C

**Data:** 09/11/2026 | **Status:** Concluído

## 1. O Roadmap da Matéria

1. **Tema 1 - Apresentação da Disciplina:** Visão geral da linguagem C e sua importância na programação de sistemas.
2. **Tema 2 - Bibliotecas e APIs para Linguagem C:** Fases de compilação, ponteiros, time.h, gráficos BGI, OpenGL/GLUT e eventos.
3. **Tema 3 - Entradas e Saídas em Linguagem C:** E/S pelo console, padronização ANSI C e UNIX/POSIX, e manipulação de arquivos de texto e binários.
4. **Tema 4 - Portas de Hardware e de Comunicação em C:** Comunicação serial RS-232, microcontroladores, Sockets TCP/UDP, HTTP e MQTT.
5. **Tema 5 - Processos e Tarefas na Linguagem C:** Criação, gerenciamento e sincronização de processos e threads.

## 2. Entrada e Saída pelo Console

A linguagem C pura é minimalista e não possui comandos de leitura ou escrita embutidos na sua sintaxe básica; tudo depende de bibliotecas como a `<stdio.h>`.

### Leitura de Caracteres: getchar() vs. getch()

| Função | Biblioteca | Como funciona |
| --- | --- | --- |
| `getchar()` | `<stdio.h>` | Padrão ANSI C. Usa buffer de linha para armazenar os caracteres digitados, mostra na tela (eco) e só processa após apertar Enter. |
| `getch()` | `<conio.h>` | Específica de Windows/DOS. Não usa buffer, não mostra o caractere na tela e captura a tecla no exato momento do clique. |

### Inspeção e Transformação de Caracteres (<ctype.h>)

* **Funções de verificação (`islower`, `isupper`):** Apenas fazem uma pergunta lógica (funcionam como um `if`) para checar se o caractere é minúsculo ou maiúsculo, sem alterar o valor original.
* **Funções de conversão (`tolower`, `toupper`):** Já convertem o caractere diretamente para minúsculo ou maiúsculo na tabela ASCII sem precisar de verificação prévia.

### Leitura de Strings e Segurança de Buffer

Usar `fgets()` no lugar de `gets()` é a regra básica de segurança para evitar estouro de memória (buffer overflow). Como o `gets()` não conhece o tamanho do vetor, se você digitar 30 caracteres em um espaço onde só cabem 20, os outros 10 invadem áreas vizinhas da memória e causam falhas graves no sistema. O `fgets()` trava um limite máximo de leitura e ignora o que passar disso.

Quando usamos o `scanf("%s", str)` comum, a leitura para no primeiro espaço em branco. Para ler textos compostos (como o nome de um livro ou de uma pessoa) usando `scanf`, aplicamos o conjunto de varredura (scanset):

```c
#include <stdio.h>

int main() {
    char titulo[100];

    printf("Digite o titulo do livro: ");
    // Espaco inicial limpa o buffer (\n anterior)
    // %99 limita a leitura segurando 1 byte para o \0
    // [^\n] le tudo (incluindo espacos) ate encontrar o Enter
    scanf(" %99[^\n]", titulo);

    printf("Livro cadastrado: %s\n", titulo);
    return 0;
}
```

### Saída de Dados e Formatação no Console

* `putchar()`: Imprime apenas 1 caractere direto na saída padrão (`stdout`).
* `puts()`: Imprime uma string inteira, consome menos processamento que o `printf` para textos simples e já adiciona a quebra de linha `\n` automaticamente no final.
* `fputs()`: Permite escolher o fluxo de saída (como `stdout` ou um arquivo), mas não pula linha sozinho.
* `printf()`: Permite formatação completa (`%d`, `%i`, `%u`, `%x`, `%f`, `%e`, `%g`, `%s`, `%c`) e retorna o número total de caracteres impressos.
* **Alinhamento no `printf`:** Usar `%8d` reserva 8 espaços alinhando o número à direita por padrão. Usar `%-8d` reserva os mesmos 8 espaços, mas força o alinhamento à esquerda.

## 3. Padronização da Linguagem C e Padrão UNIX/POSIX

### Evolução dos Padrões do C

Antes da padronização, um número inteiro ou uma função de arquivo funcionava de um jeito em um computador e falhava em outro. A linha do tempo resolveu essa falta de portabilidade:

1. **K&R C (1978):** Primeira especificação informal criada por Kernighan e Ritchie.
2. **ANSI C / C89 (1989):** Criado pelo comitê ANSI X3J11. Padronizou a linguagem garantindo portabilidade, tipos claros (`int`, `float`, etc.), prototipagem estrita de funções e bibliotecas obrigatórias como a `<stdio.h>`.
3. **ISO C / C90 (1990):** Homologação internacional idêntica ao padrão C89.
4. **C99 (1999):** Modernizou a linguagem trazendo comentários de linha única com `//`, declaração de variáveis direto dentro do laço `for (int i = 0; ...)` e bibliotecas como `<stdint.h>` e `<stdbool.h>`.

### A Relação entre C, UNIX e POSIX

O sistema operacional UNIX foi criado na Bell Labs e reescrito quase inteiramente em C por Dennis Ritchie e Ken Thompson. Com o tempo, surgiram várias versões diferentes de UNIX que perderam compatibilidade entre si.

Para resolver essa fragmentação, foi criado o padrão **POSIX** (IEEE 1003.1), que unificou as chamadas de sistema (APIs) entre sistemas da família UNIX (como Linux e macOS), garantindo que o mesmo código em C compilasse em diferentes sistemas operacionais. Já as extensões **POSIX.4** (1003.1b/c) adicionaram suporte para sistemas de tempo real, memória compartilhada, semáforos e threads.

### Chamadas de Sistema vs. Funções de Biblioteca

No manual do UNIX, as operações são divididas por camadas:

| Camada | Seção do Manual | Modo de Execução | Exemplos | Como funciona |
| --- | --- | --- | --- | --- |
| Chamadas de Sistema (System Calls) | Seção 2 | Modo Kernel | `open`, `read`, `write`, `close` | Conversam direto com o núcleo do sistema operacional sem buffer próprio na aplicação. |
| Funções de Biblioteca (Stream I/O) | Seção 3 | Modo Usuário | `fopen`, `getc`, `putc`, `fclose`, `printf` | Funções do padrão ANSI C que usam um buffer intermediário na memória e chamam a Seção 2 por baixo dos panos. |

* **Page Cache e `O_DIRECT`:** O kernel usa uma área da memória RAM (Page Cache) para guardar dados em páginas marcadas como alteradas (dirty pages) e gravar no disco depois (write-back). Se um banco de dados ou aplicação crítica precisar pular esse cache da RAM e gravar direto no disco, utiliza a flag `O_DIRECT`.

## 4. Entrada e Saída por Arquivos

### Abertura, Validação e Modos de Acesso

Para manipular arquivos na memória secundária, usamos um ponteiro do tipo `FILE*` aberto com `fopen()` e fechado obrigatoriamente com `fclose()` (que descarrega os dados pendentes do buffer e libera o arquivo).

Por segurança, sempre devemos testar se o ponteiro retornou `NULL` logo após o `fopen()`. Isso protege o programa contra falhas como arquivo inexistente, falta de permissão ou arquivo travado por outro processo, fazendo o sistema encerrar com aviso em vez de travar.

| Modo Texto | Modo Binário | Comportamento |
| --- | --- | --- |
| `"r"` | `"rb"` | Abre apenas para leitura. O arquivo precisa existir (retorna `NULL` se não achar). |
| `"w"` | `"wb"` | Abre para escrita. Cria o arquivo se não existir; se já existir, apaga todo o conteúdo anterior. |
| `"a"` | `"ab"` | Abre em modo anexação (append). Não escreve por cima dos dados existentes; escreve sempre no final ou cria o arquivo se não existir. |
| `"r+"` | `"rb+"` | Abre um arquivo já existente tanto para leitura quanto para escrita/atualização. |
| `"w+"` | `"wb+"` | Cria um arquivo do zero para leitura e escrita, apagando qualquer dado anterior. |
| `"a+"` | `"ab+"` | Abre para leitura e anexação no final do arquivo (cria se não existir). |

### Arquivos de Texto (.txt)

Gravam os dados convertidos para caracteres da tabela ASCII (no Windows, a quebra `\n` vira `\r\n`).

* **Escrita (RAM para o disco):** `fputc()` (1 caractere), `fputs()` (string sem `\n` automático) e `fprintf()` (grava texto formatado no arquivo; muito útil em logs usando `%s\n` para garantir quebra de linha limpa).
* **Leitura (Disco para a RAM):** `fgetc()` (lê caractere por caractere até `EOF`), `fscanf()` (lê formatado, mas não é recomendado para linhas de texto por falhar em espaços e buffers) e `fgets()` (a melhor opção por ler uma linha inteira do arquivo com limite de segurança).

### Arquivos Binários (.bin), Structs e Cursor Interno

Arquivos binários gravam os bytes brutos (raw bytes) exatamente como estão na memória RAM, sem converter nada para texto ASCII. Por não fazerem essa tradução de caracteres, eles ocupam menos espaço em disco e têm leitura e gravação muito mais rápidas.

As funções `fwrite()` (gravação) e `fread()` (leitura) exigem sempre 4 parâmetros:

1. Ponteiro para o endereço da variável ou `struct` na memória (`&dados`).
2. Tamanho em bytes de cada elemento individual, obtido obrigatoriamente com `sizeof()` para manter a portabilidade.
3. Quantidade de elementos que serão lidos ou gravados.
4. Ponteiro do arquivo (`FILE*`).

**Retorno de `fread` e `fwrite`:** Essas funções devolvem a **quantidade de elementos (itens)** gravados ou lidos com sucesso, e não o total de bytes. Exemplo: ao gravar um vetor de 10 inteiros (onde cada `int` ocupa 4 bytes, totalizando 40 bytes no disco), o retorno de sucesso da função será `10`.

**Controle da "Agulha" com `rewind()`:** Conforme lemos ou gravamos um arquivo, o cursor interno (a agulha de leitura/gravação) avança e fica parado no final. Se abrirmos um arquivo com `"rb+"` para ler um contador e depois gravar o valor atualizado sem voltar o cursor, ele vai escrever no final do arquivo e duplicar o tamanho em bytes. A função `rewind(fp)` rebobina a agulha de volta para o byte zero, permitindo sobrescrever o registro no lugar certo.

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int total_participantes;
} Contador;

int main() {
    // 1. Anexando participante em arquivo de texto (modo a+)
    FILE *arq_txt = fopen("participantes.txt", "a+");
    if (arq_txt == NULL) {
        printf("Erro ao abrir o arquivo de texto.\n");
        return 1;
    }

    fprintf(arq_txt, "Leonardo Martins\n");
    fclose(arq_txt);

    // 2. Atualizando contador em arquivo binario (modo rb+ ou wb+ se nao existir)
    FILE *arq_bin = fopen("contador.bin", "rb+");
    if (arq_bin == NULL) {
        arq_bin = fopen("contador.bin", "wb+");
        if (arq_bin == NULL) {
            printf("Erro ao criar o arquivo binario.\n");
            return 1;
        }
    }

    Contador c = {0};

    // Le o valor atual caso ja exista registro no arquivo
    fread(&c, sizeof(Contador), 1, arq_bin);

    // Incrementa o contador na memoria
    c.total_participantes++;

    // Rebobina a agulha para o inicio do arquivo antes de sobrescrever
    rewind(arq_bin);

    // Grava a struct atualizada e confere se 1 item inteiro foi escrito
    size_t gravados = fwrite(&c, sizeof(Contador), 1, arq_bin);
    if (gravados == 1) {
        printf("Total acumulado no binario: %d\n", c.total_participantes);
    }

    fclose(arq_bin);
    return 0;
}
```

## Pontos de Atenção

> **Pegadinha de Prova:** Cuidado para não inverter as bibliotecas e os nomes em inglês: `getchar()` é do padrão ANSI C (`<stdio.h>`) e usa buffer com Enter, enquanto `getch()` é da `<conio.h>` e captura a tecla na hora sem buffer. Nos arquivos binários, lembre que `fread` significa leitura (read) e `fwrite` significa escrita (write), enquanto `fputc` e `fscanf` são exclusivos para arquivos de texto.
> **Conceito Chave:** A eficiência do `fwrite` e `fread` em arquivos binários vem justamente do fato de gravarem os bytes direto da memória sem fazer conversão para texto ASCII. Além disso, o retorno dessas funções informa o número de itens processados (e não o número de bytes), sendo indispensável o uso do `sizeof()` no segundo parâmetro e do `rewind()` quando precisar atualizar um registro no início do arquivo.
