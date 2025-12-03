
# 📘 Projeto ESP32 – Controle de LED e Botão via MQTT

## **Alunos**: Daniel Luiz, Kauan Cavalcante e Pedro Lucas Lima

Este projeto utiliza o ESP32, conectado ao Wi-Fi no modo station, para realizar duas funções principais:

1. Enviar via MQTT o estado de um botão físico (pressionado/solto).

1. Receber comandos MQTT para ligar ou desligar um LED conectado ao pino configurado.

A comunicação é feita através do broker público HiveMQ, permitindo que o dispositivo seja controlado por outro ESP32, por um aplicativo MQTT ou pelo MQTT Explorer.

## 🗂 Funcionalidades do projeto

### ✔ Publicação MQTT (ESP → Broker)

Sempre que o botão muda de estado:

- Botão pressionado → publica "1"

- Botão solto → publica "0"

A publicação é feita no tópico:
```
embarcados/espAligaLedB
```

### ✔ Assinatura MQTT (Broker → ESP)

O ESP32 se inscreve no tópico:

```
embarcados/espAligaLedB
```
E executa:

- "1" → Liga o LED

- "0" → Desliga o LED

## ✔ LED local
O LED também pode ser acionado via tópico MQTT remoto, permitindo usar:

- Outro ESP32

- Aplicativo MQTT

- MQTT Explorer

## 🔌 Ligações elétricas

| Componente | Pino ESP32                           |
| ---------- | ------------------------------------ |
| LED        | GPIO 21                              |
| Botão      | GPIO 23 (Pull-up interno habilitado) |

O botão deve conectar **GPIO23** → **GND** ao ser pressionado.

## 📡 Configuração Wi-Fi

No código, defina SSID e senha:

```
#define ESP_WIFI_SSID      "iPhone"
#define ESP_WIFI_PASS      "123456789"
```

## 🌐 Broker MQTT

O projeto usa o broker público:

```
mqtt://broker.hivemq.com
Porta: 1883
```

Este broker não requer autenticação, ideal para testes.

## ⚙ Tópicos MQTT usados

| Função                  | Tópico                | Direção      |
| ----------------------- | --------------------- | ------------ |
| Publica estado do botão | `embarcados/espAligaLedB` | ESP → Broker |
| Recebe comando para LED | `embarcados/espAligaLedB`  | Broker → ESP |


Você pode visualizar tudo pelo MQTT Explorer.

## 🧠 Como o código funciona
### ▶ Inicialização

- Inicializa NVS

- Conecta ao Wi-Fi como station

- Ao obter IP, ativa o MQTT

- Cria a task do botão

### ▶ Task `button_led_task`

Executa continuamente:

- Lê o GPIO do botão

- Detecta mudança de estado

- Publica no tópico `embarcados/espAligaLedB`

- Registra no console

### ▶ MQTT Event Handler

Quando receber dados:
```
embarcados/espAligaLedB → "1" → liga LED  
embarcados/espAligaLedB → "0" → desliga LED
```

## 🧪 Testando com MQTT Explorer
### 1️⃣ Publicar comando para ligar o LED
- Tópico: ```embarcados/espAligaLedB```

- Mensagem: 1

LED acende na hora.

### 2️⃣ Desligar

- Mensagem: 0

### 3️⃣ Receber estado do botão
Assine:
```
embarcados/espAligaLedB
```

Você verá:
```
1  (botão pressionado)
0  (botão solto)
```

## 🤝 Usando com 2 ESP32

### ESP A (com botão)
- Publica em ```embarcados/espAligaLedB```
### ESP B (com LED)
- Inscreve em ```embarcados/espAligaLedB```
- Liga/desliga LED baseado no valor

Você só precisa trocar no ESP B:
```
esp_mqtt_client_subscribe(client, "embarcados/espAligaLedB", 0);
```

E no tratamento:
```
if(event->data[0] == '1') gpio_set_level(LED_PIN, 1);
else gpio_set_level(LED_PIN, 0);
```

Nenhum ESP precisa criar tópico — os tópicos nascem ao serem publicados no broker.

### ▶ Como compilar e enviar

1. Conectar ESP32 via USB
2. Na pasta do projeto:

```
idf.py set-target esp32
idf.py build
idf.py flash
idf.py monitor
```

### 📝 Requisitos para rodar

- ESP-IDF 5.x instalado

- Python e ambiente configurado

- Driver CP2102/CH340 instalado (dependendo do ESP32)

- Acesso a rede Wi-Fi 2.4 GHz





README.md
4 KB
