# Tema 04: Portas de Hardware e de Comunicação em C
**Data:** 25/11/2026 | **Status:** Concluído

## 1. O Roadmap da Matéria
1. **Tema 1 - Apresentação da Disciplina:** Visão geral da linguagem C e sua importância na programação de sistemas.
2. **Tema 2 - Bibliotecas e APIs para Linguagem C:** Fases de compilação, ponteiros, time.h, gráficos BGI, OpenGL/GLUT e eventos.
3. **Tema 3 - Entradas e Saídas em Linguagem C:** E/S pelo console, padronização ANSI C e UNIX/POSIX, e manipulação de arquivos de texto e binários.
4. **Tema 4 - Portas de Hardware e de Comunicação em C:** Comunicação serial RS-232, microcontroladores, PWM, sensores, Sockets TCP/UDP, HTTP e MQTT.
5. **Tema 5 - Processos e Tarefas na Linguagem C:** Criação, gerenciamento e sincronização de processos e threads.

## 2. Porta Serial, RS-232 e Microcontroladores

### Comunicação Paralela vs. Serial e UART vs. RS-232
Na comunicação paralela, vários bits são enviados ao mesmo tempo, como se existissem vários caminhos lado a lado. Na comunicação serial, existe apenas um caminho onde os bits passam em sequência, um atrás do outro.

* **UART:** É o componente físico de hardware dentro do aparelho. Ele pega os dados paralelos internos do dispositivo e converte em uma sequência serial para enviar; na recepção, pega os bits em serial e devolve para o formato paralelo original.
* **RS-232:** É o padrão físico e protocolo que define os níveis de voltagem e como esses dados trafegam no cabo. Mesmo sendo bem mais lento que USB ou Ethernet, continua muito usado em indústrias e sistemas legados pela simplicidade, baixo custo de manutenção e alta confiabilidade em ambientes difíceis (alta temperatura, vibrações e interferência eletromagnética).
* **Conector DB-9 (3 pinos essenciais):** `TXD` (envia dados), `RXD` (recebe dados) e `SG` (Signal Ground, que funciona como fio terra de referência elétrica para medir a voltagem; não passam dados por ele). Em computadores modernos sem entrada serial nativa, usam-se conversores USB-Serial.

### Porta Serial em C no Windows (`<windows.h>`)
Para conversar com uma porta COM pelo Windows em C, a WinAPI exige algumas estruturas e funções principais:
1. `CreateFile()`: Abre o acesso à porta serial (ex.: `"COM3"`).
2. `GetCommState()` e `SetCommState()`: Leem e aplicam a configuração da estrutura `DCB` (taxa de transmissão / baud rate, bits de dados, paridade e stop bits).
3. `SetCommTimeouts()`: Configura a estrutura `COMMTIMEOUTS` para definir quanto tempo esperar antes de desistir de uma leitura ou escrita.
4. `ReadFile()` e `WriteFile()`: Fazem a leitura e o envio dos bytes pela porta.
5. `CloseHandle()`: Fecha a conexão e libera a porta COM.

### Portas Digitais, Analógicas e Barramentos
Microcontroladores (como Arduino UNO e ESP32) reúnem CPU, memória e interfaces de comunicação em um único chip.

| Tipo de Porta | Como funciona na prática | Exemplos de uso |
|---|---|---|
| **Digital** | Funciona como uma lâmpada: só existem dois estados, ligado (`HIGH` / 1) ou desligado (`LOW` / 0). O Arduino UNO tem 14 pinos digitais. | Ligar LEDs, ler botões e acionar relés para ativar solenoides. |
| **Analógica** | Funciona como uma alavanca que você gira, aceitando vários níveis contínuos. No Arduino UNO (6 pinos A0 a A5), o conversor ADC transforma a tensão de entrada em um número inteiro: 0V lê `0` e 5V lê `1023`. | Sensores de temperatura, luz e pressão. |

* **Barramentos de comunicação:** Além de TX/RX (serial), microcontroladores usam **I2C** (2 fios principais: `SCL` para o clock que organiza o ritmo e `SDA` para passar os dados) e **SPI** (4 fios: `MISO`, `MOSI`, `SCK` e `SS`, exigindo um pino `SS` separado para cada dispositivo conectado).
* **Regra do `random()` no Arduino/ESP32:** O limite superior é exclusivo (`max - 1`). Para sortear um número entre 20 e 40, é obrigatório escrever `random(20, 41)` (e entre 30 e 50, `random(30, 51)`). Também não se pode somar uma string literal com inteiro usando `+` direto em C++, sendo mais seguro usar `Serial.print()` e `Serial.println()` separados.

## 3. Controle de Dispositivos, PWM e Aquisição de Dados

### Técnica PWM (Modulação por Largura de Pulso)
Uma porta digital não consegue entregar "meia voltagem", pois ela só fica ligada ou desligada. O PWM resolve isso ligando e desligando o sinal muito rápido várias vezes por segundo. Assim, um motor ou ventoinha gira em velocidade constante e um LED muda de brilho sem precisar variar a tensão real.

* **Ciclo de trabalho (Duty Cycle):** Porcentagem do tempo em que o sinal fica em nível alto (`HIGH`) dentro de um ciclo. Quanto maior o ciclo de trabalho, maior a energia média entregue (mais velocidade no motor ou mais brilho no LED).
* **Frequência:** Quantidade de ciclos por segundo; controla a suavidade da operação (evita que o LED fique piscando visivelmente ou o motor trepidando).
* **Função no Arduino:** `analogWrite(pino, valor)` aceita valores de `0` (0% ligado) até `255` (100% ligado).

### Simulação, Driver de Motor e Botões
O **Tinkercad** (da Autodesk) permite montar circuitos, escrever código C/C++ e depurar projetos virtualmente sem hardware físico, sendo acessível para iniciantes (não exige conhecimento avançado de engenharia elétrica).

* **Driver de motor DC:** Na prática fora da simulação, nunca se liga um motor direto no pino do Arduino porque a corrente alta pode queimar a placa. O Arduino envia apenas um sinal fraco de comando para o driver (transistor ou módulo ponte H), e o driver fornece a energia pesada ao motor.
* **Lógica `INPUT_PULLUP`:** Usa o resistor interno do Arduino e inverte a leitura do botão: solto lê `HIGH`, pressionado lê `LOW`.

### Sensores e Conversores ADC
Os microcontroladores usam conversores analógico-digitais (**ADCs**) integrados para traduzir sinais físicos contínuos (tensão) em números que o código consegue processar.

| Sensor | O que mede | Saída / Ligação | Detalhe importante |
|---|---|---|---|
| **PIR** | Movimento (radiação infravermelha) | Digital (0 ou 1) | Só tem dois estados: `0` sem movimento, `1` com movimento. Muito usado em segurança industrial para detectar presença em áreas perigosas. |
| **TMP36** | Temperatura | Analógica (ADC) | Gera tensão proporcional (10 mV por grau Celsius; 750 mV equivale a 25 C). Por ter muitos valores possíveis (25, 25.5, 27...), vai na porta analógica. |
| **BMP180** | Pressão barométrica e temperatura | I2C | Calcula altitude pela pressão (quanto mais alto em relação ao nível do mar, menor a pressão). No ESP32: VCC em 3.3V, GND, `SCL` no GPIO 22 e `SDA` no GPIO 21 (`<Wire.h>`, `<Adafruit_Sensor.h>`, `<Adafruit_BMP085_Unified.h>`). |
| **HC-SR04** | Distância (ultrassom) | Digital (TRIG e ECHO) | `TRIG` dispara um pulso de 10 microssegundos; `ECHO` mede o tempo de retorno com `pulseIn()`. |

Na fórmula do sensor ultrassônico HC-SR04, o resultado é dividido por 2 porque o tempo medido inclui a ida e a volta da onda sonora (velocidade do som aproximada em 340 m/s):

$$\text{distancia} = \frac{\text{tempo do eco} \times 340}{2}$$

### Regra de `pinMode()`: Sempre na Visão do Arduino
* `OUTPUT` (saída): Quando o Arduino manda energia ou sinal para fora (ex.: LED no pino 13, motor, pino `TRIG` que dispara o gatilho).
* `INPUT` (entrada): Quando o Arduino recebe uma leitura de fora (ex.: botão, pino `ECHO` que recebe o retorno do sensor).

```c
// Exemplo de controle com HC-SR04 e alerta no LED (pino 13)
#define TRIG_PIN 7
#define ECHO_PIN 8
#define LED_PIN 13

void setup() {
    pinMode(TRIG_PIN, OUTPUT); // Arduino dispara o gatilho
    pinMode(ECHO_PIN, INPUT);  // Arduino recebe o eco
    pinMode(LED_PIN, OUTPUT);  // Arduino envia energia para o LED
}

void loop() {
    digitalWrite(TRIG_PIN, LOW);
    delayMicroseconds(2);
    digitalWrite(TRIG_PIN, HIGH);
    delayMicroseconds(10);
    digitalWrite(TRIG_PIN, LOW);

    long duration = pulseIn(ECHO_PIN, HIGH);
    float distanceCm = (duration * 0.034) / 2.0;

    if (distanceCm < 50.0) {
        digitalWrite(LED_PIN, HIGH);
        delay(100);
        digitalWrite(LED_PIN, LOW);
        delay(100);
    }
}
```

## 4. Sockets e Cliente/Servidor TCP e UDP

### Conceito de Sockets e Posição na Rede
Sockets são pontos finais (endpoints) que permitem comunicação bidirecional (envio e recebimento de dados) entre programas na mesma máquina ou em máquinas diferentes pela rede.

* **IP + Porta:** O **Endereço IP** identifica o dispositivo na rede; o **Número de Porta** direciona os dados para o aplicativo ou serviço correto dentro daquele dispositivo receptor.
* **Biblioteca vs. API:** Uma biblioteca é uma coleção de código e funções prontas; uma API é o contrato (conjunto de regras e assinaturas) que define como programas se comunicam com o sistema ou entre si.
* **Camadas de rede:** A API de sockets fica no Kernel fazendo a ponte entre a **Camada de Aplicação** e a **Camada de Transporte** (TCP/UDP). Ela não substitui a camada de transporte e não atua diretamente nas camadas física ou de enlace.
* **POSIX vs. Winsock:** Ambos derivam dos sockets BSD. O padrão POSIX é nativo em Linux/Unix/macOS, enquanto a Winsock (`<winsock2.h>`, linkada com `libwsock32.a` no Dev-C++) traz essa interface para o Windows, exigindo inicialização com `WSAStartup()` e encerramento com `WSACleanup()`.

### Ordem de Bytes na Rede (Endianness)
Arquiteturas de processadores diferentes guardam bytes na memória em ordens opostas:
* **Big-endian:** Guarda o byte mais significativo (MSB) no menor endereço. É o padrão oficial da rede (Network Byte Order).
* **Little-endian:** Guarda o byte menos significativo (LSB) no menor endereço (usado na maioria dos PCs x86/x64).

Para que máquinas diferentes se entendam sem corromper números de porta ou endereços IP, a API em C oferece funções de conversão obrigatórias:
* `htons()` e `ntohs()`: Convertem inteiros de 16 bits (usados no número da **porta**), de Host para Network e vice-versa.
* `htonl()` e `ntohl()`: Convertem inteiros de 32 bits (usados no endereço **IPv4**), de Host para Network e vice-versa.

### TCP vs. UDP e Funções em C

| Característica | TCP (`SOCK_STREAM`) | UDP (`SOCK_DGRAM`) |
|---|---|---|
| **Conexão** | Orientado a conexão (estabelece sessão antes de enviar). | Sem conexão (envia datagramas direto). |
| **Confiabilidade** | Alta: garante entrega, ordem dos pacotes e reenvio em caso de erro. | Sem garantia de entrega ou ordem. |
| **Velocidade / Latência** | Mais lento devido ao controle e confirmações. | Muito rápido e com baixa latência. |
| **Casos de uso** | Navegação Web (HTTP), transferência de arquivos, e-mail. | Jogos on-line, streaming ao vivo, telemetria de IoT. |
| **Funções de troca de dados** | `send()` e `recv()` | `sendto()` e `recvfrom()` |

**Ciclo básico das funções da API de Sockets:**
* **Servidor TCP:** `socket()` -> `bind()` (associa IP e porta) -> `listen()` (aguarda conexões) -> `accept()` (aceita um cliente) -> `recv()` / `send()` -> `close()` ou `closesocket()`.
* **Cliente TCP:** `socket()` -> `connect()` (conecta ao servidor) -> `send()` / `recv()` -> `close()` ou `closesocket()`.
* **UDP:** Não usa `listen()` nem `accept()`; o servidor faz `socket()` -> `bind()` -> `recvfrom()` / `sendto()`, e o cliente usa `socket()` -> `sendto()` / `recvfrom()`.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <winsock2.h>

// No Dev-C++ / MinGW, linkar com a biblioteca -lwsock32 ou -lws2_32
int main() {
    WSADATA wsa;
    SOCKET sock;
    struct sockaddr_in servidor;
    char mensagem[64] = "Ping do cliente C";
    char resposta[128];

    // 1. Inicializa a Winsock no Windows
    if (WSAStartup(MAKEWORD(2, 2), &wsa) != 0) {
        printf("Falha ao inicializar Winsock.\n");
        return 1;
    }

    // 2. Cria o socket TCP (IPv4 + SOCK_STREAM)
    sock = socket(AF_INET, SOCK_STREAM, 0);
    if (sock == INVALID_SOCKET) {
        printf("Erro ao criar socket.\n");
        WSACleanup();
        return 1;
    }

    // 3. Configura IP e porta convertendo para ordem de rede (Big-endian)
    servidor.sin_family = AF_INET;
    servidor.sin_addr.s_addr = inet_addr("127.0.0.1");
    servidor.sin_port = htons(8080);

    // 4. Conecta ao servidor
    if (connect(sock, (struct sockaddr *)&servidor, sizeof(servidor)) < 0) {
        printf("Erro de conexao.\n");
        closesocket(sock);
        WSACleanup();
        return 1;
    }

    // 5. Envia e recebe dados
    send(sock, mensagem, (int)strlen(mensagem), 0);
    int bytes = recv(sock, resposta, sizeof(resposta) - 1, 0);
    if (bytes > 0) {
        resposta[bytes] = '\0';
        printf("Servidor respondeu: %s\n", resposta);
    }

    // 6. Encerra o socket e limpa a Winsock
    closesocket(sock);
    WSACleanup();
    return 0;
}
```

## 5. Protocolos HTTP e MQTT para IoT

### Arquitetura Cliente/Servidor HTTP
O HTTP (Hypertext Transfer Protocol) define a comunicação da Web no modelo requisição-resposta. O fluxo completo ao acessar uma URL segue esta sequência:

1. **Conexão TCP:** O cliente (navegador) abre uma conexão TCP com o servidor na **porta 80** (HTTP) ou **porta 443** (HTTPS com criptografia SSL/TLS).
2. **Requisição HTTP:** O cliente envia o método (`GET`, `POST`, etc.), o caminho do recurso desejado e os cabeçalhos da requisição.
3. **Processamento:** O servidor recebe o pedido e prepara o recurso (HTML, JSON, imagem ou comando em um microcontrolador).
4. **Resposta HTTP:** O servidor devolve a linha de status (como `200 OK` ou `404 Not Found`), os **cabeçalhos** (que incluem informações do servidor, tipo de conteúdo e diretivas de cache) e o corpo da resposta.
5. **Gerenciamento e renderização:** No HTTP/1.1, a conexão TCP pode ser reaproveitada (persistente) para baixar arquivos extras (CSS, JS, imagens) enquanto o navegador renderiza a página.

* **4 vantagens principais do HTTP:** Escalabilidade, flexibilidade, facilidade de manutenção e eficiência.
* **Uso prático em embarcados:** Um ESP32 pode rodar um pequeno servidor web HTTP na rede local, servindo uma página HTML/CSS com botões para ligar ou desligar um LED direto pelo navegador do celular ou PC.

### Arquitetura MQTT para IoT
O MQTT (Message Queuing Telemetry Transport) é um protocolo leve criado para dispositivos com pouco poder de processamento, bateria limitada e redes instáveis. Diferente do HTTP tradicional, ele usa o modelo **Publicador / Subscritor** (Publish/Subscribe):

* **Broker:** Servidor central que recebe todas as mensagens, filtra por tópicos e redistribui para os interessados. Os dispositivos não precisam conhecer o IP uns dos outros.
* **Clientes:** Dispositivos ou aplicativos que podem publicar dados em um tópico, assinar (subscrever) tópicos para receber atualizações, ou fazer ambos.
* **Tópicos:** Canais organizados de forma hierárquica parecidos com caminhos de pasta (ex.: `emqx/esp32` ou `casa/sala/temperatura`).

#### Níveis de Qualidade de Serviço (QoS)
| Nível | Garantia de entrega | Comportamento |
|---|---|---|
| **QoS 0** | No máximo uma vez | Envia sem esperar confirmação ("dispara e esquece"). Mais rápido, mas pode perder pacotes. |
| **QoS 1** | Ao menos uma vez | Garante que a mensagem chegue com confirmação, mas pode gerar duplicatas se a rede falhar. |
| **QoS 2** | Exatamente uma vez | Processo de confirmação em etapas que evita perda e duplicação. Mais seguro, porém mais lento. |

* **5 vantagens do MQTT para IoT:** Eficiência de banda larga, baixo consumo de energia (compatível com *sleep mode*), alta escalabilidade, segurança (autenticação e SSL/TLS) e flexibilidade de integração.
* **Uso prático em embarcados:** O ESP32 lê a temperatura de um sensor BMP180 via I2C e usa a biblioteca `PubSubClient` para enviar os dados via `client.publish()` para um tópico em um broker (como o broker público EMQX), disparando alertas quando o valor passa de 28 C.

## Pontos de Atenção
> **Pegadinha de Prova:** O RS-232 não se destaca por alta velocidade de transferência, mas pela confiabilidade em ambientes industriais severos. No Tinkercad, a ferramenta é voltada para aprendizado e iniciantes em C/C++, não exigindo conhecimento avançado de engenharia elétrica. Na resposta HTTP, os cabeçalhos incluem sim informações do servidor e regras de cache. Além disso, a API de sockets faz a ponte entre a Camada de Aplicação e a Camada de Transporte (não substitui a camada de transporte nem acessa a camada física diretamente).
> **Conceito Chave:** No Arduino/ESP32, `pinMode()` é sempre definido na perspectiva da placa (`TRIG` e LED são `OUTPUT`; `ECHO` é `INPUT`), `INPUT_PULLUP` inverte a leitura do botão (pressionado = `LOW`) e `random(min, max)` nunca inclui o valor `max`. Em redes com C, sockets são bidirecionais, a porta direciona ao aplicativo dentro do dispositivo (não a outro dispositivo), e os valores de porta e IP devem ser convertidos para Big-endian com `htons()` e `htonl()`.