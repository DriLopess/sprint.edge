# sprint.edge
# README - Medidor de Energia IoT com Docker Compose

Este é um projeto que demonstra a implementação de um medidor de energia IoT utilizando um medidor de corrente ACS712 de 30A, um microcontrolador ESP32, e um conversor de energia para 5V. Além do IoT, projeto faz uso de uma API para a criação e visualização dos dados no dashboard. 

### Hardware:
- Dispositivos IoT (medidores de energia) - um para cada tomada a ser monitorada.
- ESP32.
- Medidor de corrente ACS712 de 30A.
- Conversor de energia para 5V.
- Cabos e conexões adequadas.

## Funcionamento do Medidor de Energia

O medidor de energia (MeterWatt) é colocado em cada tomada da casa para medir o consumo de energia em tempo real. Ele funciona da seguinte maneira:

1. **Conexão de Hardware**:
   - A porta VCC do ACS712 está conectada à porta VIN do ESP32.
   - O GND do ACS712 está conectado ao GND do ESP32.
   - A porta OUT do ACS712 está conectada à porta 34 do ESP32.

2. **Código do Processo**:

#include <WiFi.h>
#include <PubSubClient.h> // Importa a Biblioteca PubSubClient
#include <HTTPClient.h>
#include <ArduinoJson.h>


//defines:
//defines de id mqtt e tópicos para publicação e subscribe denominado TEF(Telemetria e Monitoramento de Equipamentos)
#define TOPICO_SUBSCRIBE    "/TEF/meterwatt001/cmd"        //tópico MQTT de escuta
#define TOPICO_PUBLISH      "/TEF/meterwatt001/attrs"      //tópico MQTT de envio de informações para Broker
#define TOPICO_PUBLISH_2    "/TEF/meterwatt001/attrs/p"    //tópico MQTT de envio de informações para Broker
                                                      //IMPORTANTE: recomendamos fortemente alterar os nomes
                                                      //            desses tópicos. Caso contrário, há grandes
                                                      //            chances de você controlar e monitorar o ESP32
                                                      //            de outra pessoa.
#define ID_MQTT  "fiware_meterwatt001"      //id mqtt (para identificação de sessão)
                                 //IMPORTANTE: este deve ser único no broker (ou seja, 
                                 //            se um client MQTT tentar entrar com o mesmo 
                                 //            id de outro já conectado ao broker, o broker 
                                 //            irá fechar a conexão de um deles).
                                 // o valor "n" precisa ser único!

#include "EmonLib.h"                        // inclui a biblioteca
EnergyMonitor emon1;                        // Cria uma instância
 
#define   SAMPLING_TIME     0.0001668649    // intervalo de amostragem 166,86us
#define   LINE_FREQUENCY    60              // frequencia 60Hz Brasil
 
#define   VOLTAGE_AC        127.00          // 127 Volts
#define   ACS_MPY           12.46           // ganho/calibracao da corrente
 
double Irms = 0;
float potencia;                                

// WIFI
const char* SSID = "CASAS 2.4G"; // SSID / nome da rede WI-FI que deseja se conectar
const char* PASSWORD = "ass0607100"; // Senha da rede WI-FI que deseja se conectar
const char* serverAddress = "http://192.168.15.13:5000/receber_dados";
// MQTT
const char* BROKER_MQTT = "46.17.108.113"; //URL do broker MQTT que se deseja utilizar
int BROKER_PORT = 1883; // Porta do Broker MQTT
 
int D4 = 2;

//Variáveis e objetos globais
WiFiClient espClient; // Cria o objeto espClient
PubSubClient MQTT(espClient); // Instancia o Cliente MQTT passando o objeto espClient
char EstadoSaida = '0';  //variável que armazena o estado atual da saída
  
//Prototypes
void initSerial();
void initWiFi();
void initMQTT();
void reconectWiFi(); 
void mqtt_callback(char* topic, byte* payload, unsigned int length);
void VerificaConexoesWiFIEMQTT(void);
void InitOutput(void);
 
/* 
 *  Implementações das funções
 */
void setup() 
{
    //inicializações:
    emon1.current(34, ACS_MPY); 
    InitOutput();
    initSerial();
    initWiFi();
    initMQTT();
    delay(5000);
    MQTT.publish(TOPICO_PUBLISH, "s|on");
}
  
//Função: inicializa comunicação serial com baudrate 115200 (para fins de monitorar no terminal serial 
//        o que está acontecendo.
//Parâmetros: nenhum
//Retorno: nenhum
void initSerial() 
{
    Serial.begin(115200);
}
 
//Função: inicializa e conecta-se na rede WI-FI desejada
//Parâmetros: nenhum
//Retorno: nenhum
void initWiFi() 
{
    delay(10);
    Serial.println("------Conexao WI-FI------");
    Serial.print("Conectando-se na rede: ");
    Serial.println(SSID);
    Serial.println("Aguarde");
     
    reconectWiFi();
}
  
//Função: inicializa parâmetros de conexão MQTT(endereço do 
//        broker, porta e seta função de callback)
//Parâmetros: nenhum
//Retorno: nenhum
void initMQTT() 
{
    MQTT.setServer(BROKER_MQTT, BROKER_PORT);   //informa qual broker e porta deve ser conectado
    MQTT.setCallback(mqtt_callback);            //atribui função de callback (função chamada quando qualquer informação de um dos tópicos subescritos chega)
}
  
//Função: função de callback 
//        esta função é chamada toda vez que uma informação de 
//        um dos tópicos subescritos chega)
//Parâmetros: nenhum
//Retorno: nenhum
void mqtt_callback(char* topic, byte* payload, unsigned int length) 
{
    String msg;
     
    //obtem a string do payload recebido
    for(int i = 0; i < length; i++) 
    {
       char c = (char)payload[i];
       msg += c;
    }
    
    Serial.print("- Mensagem recebida: ");
    Serial.println(msg);
    
    //toma ação dependendo da string recebida:
    //verifica se deve colocar nivel alto de tensão na saída D0:
    //IMPORTANTE: o Led já contido na placa é acionado com lógica invertida (ou seja,
    //enviar HIGH para o output faz o Led apagar / enviar LOW faz o Led acender)
    if (msg.equals("lamp001@on|"))
    {
        digitalWrite(D4, HIGH);
        EstadoSaida = '1';
    }
 
    //verifica se deve colocar nivel alto de tensão na saída D0:
    if (msg.equals("lamp001@off|"))
    {
        digitalWrite(D4, LOW);
        EstadoSaida = '0';
    }
     
}
  
//Função: reconecta-se ao broker MQTT (caso ainda não esteja conectado ou em caso de a conexão cair)
//        em caso de sucesso na conexão ou reconexão, o subscribe dos tópicos é refeito.
//Parâmetros: nenhum
//Retorno: nenhum
void reconnectMQTT() 
{
    while (!MQTT.connected()) 
    {
        Serial.print("* Tentando se conectar ao Broker MQTT: ");
        Serial.println(BROKER_MQTT);
        if (MQTT.connect(ID_MQTT)) 
        {
            Serial.println("Conectado com sucesso ao broker MQTT!");
            MQTT.subscribe(TOPICO_SUBSCRIBE); 
        } 
        else
        {
            Serial.println("Falha ao reconectar no broker.");
            Serial.println("Havera nova tentatica de conexao em 2s");
            delay(2000);
        }
    }
}
  
//Função: reconecta-se ao WiFi
//Parâmetros: nenhum
//Retorno: nenhum
void reconectWiFi() 
{
    //se já está conectado a rede WI-FI, nada é feito. 
    //Caso contrário, são efetuadas tentativas de conexão
    if (WiFi.status() == WL_CONNECTED)
        return;
         
    WiFi.begin(SSID, PASSWORD); // Conecta na rede WI-FI
     
    while (WiFi.status() != WL_CONNECTED) 
    {
        delay(100);
        Serial.print(".");
    }
   
    Serial.println();
    Serial.print("Conectado com sucesso na rede ");
    Serial.print(SSID);
    Serial.println("IP obtido: ");
    Serial.println(WiFi.localIP());
}
 
//Função: verifica o estado das conexões WiFI e ao broker MQTT. 
//        Em caso de desconexão (qualquer uma das duas), a conexão
//        é refeita.
//Parâmetros: nenhum
//Retorno: nenhum
void VerificaConexoesWiFIEMQTT(void)
{
    if (!MQTT.connected()) 
        reconnectMQTT(); //se não há conexão com o Broker, a conexão é refeita
     
     reconectWiFi(); //se não há conexão com o WiFI, a conexão é refeita
}
 
//Função: envia ao Broker o estado atual do output 
//Parâmetros: nenhum
//Retorno: nenhum
void EnviaEstadoOutputMQTT(void)
{
    if (EstadoSaida == '1')
    {
      MQTT.publish(TOPICO_PUBLISH, "s|on");
      Serial.println("- Led Ligado");
    }
    if (EstadoSaida == '0')
    {
      MQTT.publish(TOPICO_PUBLISH, "s|off");
      Serial.println("- Led Desligado");
    }
    Serial.println("- Estado do LED onboard enviado ao broker!");
    delay(1000);
}
 
//Função: inicializa o output em nível lógico baixo
//Parâmetros: nenhum
//Retorno: nenhum
void InitOutput(void)
{
    //IMPORTANTE: o Led já contido na placa é acionado com lógica invertida (ou seja,
    //enviar HIGH para o output faz o Led apagar / enviar LOW faz o Led acender)
    pinMode(D4, OUTPUT);
    digitalWrite(D4, HIGH);
    
    boolean toggle = false;

    for(int i = 0; i <= 10; i++)
    {
        toggle = !toggle;
        digitalWrite(D4,toggle);
        delay(200);           
    }
}
 
 
//programa principal
void loop() 
{   
    const int potPin = 34;
    
    char msgBuffer[5];
    //garante funcionamento das conexões WiFi e ao broker MQTT
    VerificaConexoesWiFIEMQTT();
 
    //envia o status de todos os outputs para o Broker no protocolo esperado
    EnviaEstadoOutputMQTT();

    Irms = emon1.calcIrms(1996);  // Calculate Irms only
    Serial.println(Irms);
    potencia = Irms * VOLTAGE_AC;
    Serial.println(potencia);
    dtostrf(potencia, 4, 2, msgBuffer);
    MQTT.publish(TOPICO_PUBLISH_2,msgBuffer);
     StaticJsonDocument<200> doc;
    doc["sensor1"] = 123;  // Substitua pelos dados do sensor 1
    doc["sensor2"] = 456;  // Substitua pelos dados do sensor 2

    String payload;
    serializeJson(doc, payload);

  // Envia os dados para o Flask
    HTTPClient http;
    http.begin(serverAddress);
    http.addHeader("Content-Type", "application/json");

    int httpResponseCode = http.POST(payload);

    if (httpResponseCode > 0) {
      Serial.print("HTTP Response code: ");
      Serial.println(httpResponseCode);
    } else {
      Serial.print("Error on HTTP request: ");
      Serial.println(httpResponseCode);
    }

    http.end();

    delay(5000);  // Envie os dados a cada 10 segundos (ou conforme necessário)
  
    //keep-alive da comunicação com broker MQTT
    MQTT.loop();
}
## Instruções de Uso

1. **Preparação do Hardware**:
   - Conecte o ACS712 ao ESP32 conforme descrito na seção "Conexão de Hardware".
   - Certifique-se de que os dispositivos IoT estejam instalados em cada tomada.

2. **Configuração do Software**:
   - Certifique-se de ter o Docker Compose instalado na sua máquina.

3. **Execução do Projeto IoT**:
   - Clone este repositório git https://github.com/fabiocabrini/fiware
   - Use o Docker Compose para iniciar os serviços do backend e frontend da aplicação IoT.

   ```bash
   docker-compose up -d
   ```

4. **Acessando o Frontend**:
   - Para obter acesso ao Frontend, será necessário executar a API localmente em Python com o seguinte codigo:
   - from flask import Flask
from flask_cors import CORS
import requests
import json
import threading
import time

app = Flask(__name__)
app.config['JSON_SORT_KEYS'] = False
CORS(app, resources={r"/get": {"origins": "http://localhost:5173"}})

reais = 0.0
watts = 0.0
wattsAcumulativo = 0.0
reaisAcumulativo = 0.0

def atualizar_dados():
    global wattsAcumulativo
    global reaisAcumulativo
    global watts
    global reais

    while True:
        url = "http://46.17.108.113:1026/v2/entities/urn:ngsi-ld:Meterwatt:001"
        payload = {}
        headers = {
            'Accept': 'application/json',
            'fiware-service': 'smart',
            'fiware-servicepath': '/',
            "Access-Control-Allow-Origin": "http://localhost:5173"
        }

        response = requests.request("GET", url, headers=headers, data=payload)
        data = json.loads(response.text)

        if 'pot' in data and 'value' in data['pot']:
            watts = data['pot']['value']
            transform = watts * 0.0012
            reais = transform / 1000
            reaisAcumulativo += reais
            wattsAcumulativo += watts

        time.sleep(10)

atualizacao_thread = threading.Thread(target=atualizar_dados)
atualizacao_thread.daemon = True
atualizacao_thread.start()

@app.route('/get', methods=['GET'])
def leitura():
    global reais
    global watts
    global reaisAcumulativo
    global wattsAcumulativo

    return json.dumps({
        'reais': reais,
        'watts': watts,
        'reais_acumulativo': reaisAcumulativo,
        'watts_acumulativo': wattsAcumulativo
    })

if __name__ == "__main__":
    app.run(host='0.0.0.0', debug=True, port='5000')

5. **Visualizando os Dados**:
   - Na interface do frontend, você poderá visualizar os dados de consumo de energia em tempo real.
