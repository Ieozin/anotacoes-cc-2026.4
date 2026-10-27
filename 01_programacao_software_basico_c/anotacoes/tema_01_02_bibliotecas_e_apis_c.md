# Tema 01 e 02: Fundamentos da Linguagem C, Bibliotecas e APIs
**Data:** 27/10/2026 | **Status:** Concluído

## 1. O Roadmap da Matéria
1. **Tema 1 - Apresentação da Disciplina:** Visão geral da linguagem C e sua importância para a programação de sistemas e software básico.
2. **Tema 2 - Bibliotecas e APIs para Linguagem C:** Fases de compilação, ponteiros, time.h, gráficos BGI, pipeline do OpenGL/GLUT e controle de eventos.
3. **Tema 3 - Entradas e Saídas em Linguagem C:** Manipulação de arquivos, fluxos de dados de baixo nível e buffers de I/O.
4. **Tema 4 - Portas de Hardware e de Comunicação em C:** Acesso direto a portas físicas, protocolos e controle de dispositivos.
5. **Tema 5 - Processos e Tarefas na Linguagem C:** Criação, gerenciamento e sincronização de processos e threads no sistema operacional.

## 2. Etapas de Compilação e Tipos Básicos
O processo que transforma o código-fonte escrito em C em um binário executável passa por 4 etapas sequenciais:

1. **Pré-processamento:** Lê todas as diretivas iniciadas por `#` (como `#include` e `#define`), remove os comentários do texto e expande o código-fonte bruto.
2. **Compilação:** Traduz o código expandido para a linguagem assembly intermediária da arquitetura alvo.
3. **Montagem (Assembler):** Converte as instruções em assembly para código de máquina puro, gerando os arquivos objeto (`.obj` ou `.o`).
4. **Link-edição (Linker):** Combina os arquivos objeto com as bibliotecas do sistema e dependências externas para gerar o executável final (`.exe` ou binário de sistema).

### Diferença entre #define e const
| Recurso | Como funciona |
|---|---|
| `#define` | Substituição direta de texto realizada na fase de pré-processamento. Não aloca memória de variável nem realiza verificação de tipos. |
| `const` | Declaração de variável com tipo fixo protegida em memória contra escrita, validada com rigor pelo compilador. |

### Operador de Resto (%)
O operador `%` retorna o resto da divisão inteira. Por exemplo, a operação $14 \pmod 4$ resulta em $2$.
Esse operador exige que ambos os operandos sejam obrigatoriamente do tipo inteiro (`int`, `char`, `short`, `long`). O compilador rejeita expressões que utilizem variáveis decimais de ponto flutuante (`float` ou `double`).

### Classes de Armazenamento
- `auto`: Padrão para variáveis locais dentro de funções ou blocos; o espaço é descartado ao término do bloco.
- `static`: Mantém o valor salvo entre chamadas consecutivas da mesma função e restringe a visibilidade da variável ou função ao arquivo local.
- `extern`: Sinaliza ao compilador que a variável ou função foi definida em outro arquivo fonte do projeto.
- `register`: Sugere ao compilador alocar a variável diretamente em um registrador livre da CPU para acelerar acessos repetitivos.

## 3. Ponteiros e Alocação Dinâmica de Memória
Um ponteiro é uma variável cujo conteúdo é um endereço de memória física ou virtual. Quando usamos `p`, estamos referenciando o endereço armazenado; ao usar `*p`, fazemos a desreferenciação, acessando ou alterando o valor contido naquele endereço.

A função `free()` só pode ser executada sobre ponteiros retornados por chamadas de alocação dinâmica (`malloc`, `calloc` ou `realloc`). Tentar liberar memória estática ou ponteiros não alocados corrompe a memória da aplicação.

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int *ptr = NULL;
    int tamanho = 5;

    // Alocacao dinamica de memoria para 5 inteiros
    ptr = (int *) malloc(tamanho * sizeof(int));

    // Validacao obrigatoria do retorno
    if (ptr == NULL) {
        printf("Falha na alocacao de memoria.\n");
        return 1;
    }

    // Aritmetica de ponteiros e desreferenciacao
    for (int i = 0; i < tamanho; i++) {
        *(ptr + i) = (i + 1) * 10;
        printf("Posicao %d: %d (Endereco: %p)\n", i, *(ptr + i), (void *)(ptr + i));
    }

    // Liberacao do espaco alocado para evitar vazamento de memoria (memory leak)
    free(ptr);
    ptr = NULL;

    return 0;
}
```

### Otimização em Laços (Primalidade)
Ao verificar se um número inteiro é primo, testar divisores até o próprio valor é custoso computacionalmente. A verificação deve ocorrer apenas enquanto $i \times i \le \text{num}$ (limite equivalente a $\sqrt{\text{num}}$). 

Se a condição `num % i == 0` for verdadeira em qualquer passo, a divisão foi exata e o número não é primo. É mandatório incrementar o contador do laço a cada iteração (`num++` ou `i++`) para evitar loops infinitos.

## 4. Manipulação Temporal e Gráficos BGI
### time.h
- `time_t`: Tipo numérico que armazena a contagem total de tempo em segundos decorridos desde o Unix Epoch ($01/01/1970$ às $00:00:00$ UTC).
- `struct tm`: Estrutura com os campos de data e hora decompostos:
  - `tm_sec`: Segundos ($0$ a $59$).
  - `tm_min`: Minutos ($0$ a $59$).
  - `tm_hour`: Horas ($0$ a $23$).
  - `tm_mday`: Dia do mês ($1$ a $31$).
  - `tm_mon`: Mês do ano ($0$ a $11$, onde janeiro é $0$).
  - `tm_year`: Anos contados a partir de $1900$.
- **Buffer de memória:** Vetor temporário alocado para que funções como `strftime` possam preencher sequências de caracteres formatadas com base na estrutura de tempo.

### Gráficos com graphics.h (BGI)
A biblioteca gráfica BGI foi desenvolvida originalmente pela Borland e é utilizada para fins de aprendizado por permitir desenho direto na tela com comandos simples.

- `initgraph` e `closegraph`: Inicializam o subsistema gráfico e liberam a memória ao fechar a janela.
- `getmaxx` e `getmaxy`: Informam a resolução máxima horizontal e vertical da tela.
- `cleardevice`: Limpa o quadro anterior. Sem a chamada periódica dessa rotina no laço de animação, os desenhos anteriores permanecem visíveis, gerando um efeito de borrão contínuo (ghosting).
- **Geração de números pseudoaleatórios:** `srand(time(NULL))` inicializa a semente randômica com o relógio do sistema. A expressão `(rand() % 8 - 4)` gera números inteiros no intervalo de $[-4, 3]$, permitindo variações equilibradas de trajetória para a bola em jogos 2D.

## 5. Gráficos 3D com OpenGL e GLUT
### OpenGL como Máquina de Estados
O OpenGL opera segundo o modelo de máquina de estados finitos. Uma instrução de configuração de cor (`glColor3f`) ou de material permanece ativa para todas as primitivas geométricas desenhadas posteriormente, até que uma nova instrução altere esse estado.

Trocas frequentes de contexto e estado exigem grande esforço de processamento na GPU. A boa prática consiste em agrupar primitivas com as mesmas propriedades gráficas antes de solicitar a alteração de estado.

### Biblioteca GLUT
A GLUT fornece abstrações para gerenciar janelas, laços de eventos e periféricos sem exigir comandos específicos de sistemas operacionais. Para rodar em ambiente Windows, o arquivo `glut32.dll` precisa estar disponível na pasta do executável ou nos diretórios do sistema (`System32` ou `SysWOW64`).

Funções centrais de controle:
- `glutInit`: Inicializa os parâmetros de terminal da biblioteca.
- `glutInitDisplayMode`: Configura canais de cor (RGB/RGBA), buffer de profundidade e buffer duplo.
- `glutCreateWindow`: Cria a janela física de renderização.
- `glutDisplayFunc`, `glutReshapeFunc`, `glutIdleFunc`: Registram funções de retorno (callbacks) para desenhar a cena, tratar redimensionamentos de tela e processar animações de fundo.
- `glutMainLoop`: Transfere o controle da aplicação para o loop contínuo de processamento gráfico.

### Estruturas do Pipeline Gráfico 3D
| Recurso | Função prática |
|---|---|
| Buffer Duplo (`GLUT_DOUBLE` e `glutSwapBuffers`) | Renderiza o novo quadro no buffer traseiro oculto e realiza a troca com o buffer frontal em um único passo, prevenindo cintilações (flickering). |
| Z-Buffer (`GL_DEPTH_TEST`) | Executa o teste de profundidade descartando pixels que estejam ocultos por superfícies mais próximas da câmera. |
| Matrizes de Câmera | A matriz `GL_PROJECTION` configura projeções como `gluPerspective`; a matriz `GL_MODELVIEW` define a posição dos objetos e da câmera (`gluLookAt`). A instrução `glLoadIdentity()` deve ser chamada para restaurar a matriz identidade antes de aplicar novas transformações. |

## 6. Tratamento de Eventos e Lógica de Aplicações Interativas
### Interrupções vs. Eventos
- **Interrupções:** São sinais emitidos diretamente pelo hardware que forçam o processador a interromper a rotina corrente para executar um tratador imediato de alta prioridade.
- **Eventos:** Notificações enfileiradas pelo sistema operacional que são extraídas e despachadas sequencialmente pelo loop de mensagens da aplicação (como `GetMessage` e `DispatchMessage` no Windows).

### Callbacks de Entrada no GLUT
- **Teclas alfanuméricas:** Mapeadas pela função `glutKeyboardFunc`, recebendo códigos da tabela ASCII convencional (caracteres de texto, espaço e Enter).
- **Teclas especiais:** Mapeadas obrigatoriamente pela função `glutSpecialFunc`, sendo indispensável para capturar as setas de direção (`GLUT_KEY_UP`, `GLUT_KEY_DOWN`, etc.) e teclas de função (`F1` a `F12`).
- **Dispositivo apontador (Mouse):**
  - `glutMouseFunc`: Trata cliques e solturas de botões.
  - `glutMotionFunc`: Rastreia o movimento enquanto algum botão está pressionado.
  - `glutPassiveMotionFunc`: Rastreia a movimentação livre do cursor sobre a janela.

### Temporizadores em Jogos (Telejogo / Pong)
No framework GLUT, a atualização física de posições utiliza temporizadores registrados por `glutTimerFunc`. A velocidade aparente dos elementos na tela é inversamente proporcional ao intervalo em milissegundos configurado no temporizador: ao aumentar o tempo de espera do timer, a frequência de quadros cai e a velocidade da bola diminui.

## Pontos de Atenção
> **Pegadinha de Prova:** O tipo `time_t` contabiliza o tempo estritamente em segundos desde 1970, nunca em milissegundos. Além disso, o operador de módulo (`%`) gera erro de compilação imediato se for utilizado com tipos de ponto flutuante (`float` ou `double`).
> **Conceito Chave:** No GLUT, teclas direcionais do teclado não disparam callbacks registrados com `glutKeyboardFunc`, exigindo obrigatoriamente `glutSpecialFunc`. No pipeline OpenGL, cada troca de estado possui custo computacional na GPU, devendo-se agrupar elementos de mesma cor e textura.
