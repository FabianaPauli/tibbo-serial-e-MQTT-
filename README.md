# 🏭 Sistema de Monitoramento de Esteira (Tibbo/Node-RED)

Este repositório contém o fluxo do Node-RED para monitorar e simular os dados de uma esteira de produção. O fluxo rastreia produtos, controla o status da máquina e publica métricas em tópicos MQTT para consumo pelo seu dashboard.

---

## ⚙️ Visão Geral do Fluxo

O fluxo é acionado por um nó `Inject` (Simulação/Reset) e executa três tarefas principais em paralelo, usando variáveis de contexto para rastrear a contagem e o status da máquina.

### Diagrama do Fluxo (Visão Geral)



---

## 🔗 Mapeamento e Tópicos MQTT

Os dados cruciais da esteira são publicados nos seguintes tópicos, que são a espinha dorsal do sistema de monitoramento:

| Tópico MQTT | Tipo de Dado | Dado Publicado (Exemplo) | Função |
| :--- | :--- | :--- | :--- |
| `Topico/sn` | `string` | `"teste"` | **Serial Number** (ou Lote) do produto lido. |
| `Topico/time` | `string` | `"2025-11-21 04:31:22 PM"` | Tempo exato em que a leitura do item foi feita. |
| `Topico/total/produtos` | `number` | `3` | **Contagem total de produtos** processados (usado no nó Gauge). |
| `Topico/total/soma` | `number` | `0` | Soma total dos itens (variável de controle interna). |
| `Topico/status/maquina` | `number` | `1` ou `0` | **Status da máquina** (`1` = Ligada, `0` = Desligada). |

---

## 🛠️ Descrição Detalhada dos Componentes Chave

O fluxo usa nós de função e nós `Change` (Mudar) para manipular e armazenar dados, e nós **MQTT Publish** para enviar informações ao broker.

### 1. Publicação de Dados e Identificação (Caminho Superior)

* **`[Inject]` (payload: `teste`)**: Inicia o ciclo de produção.
* **`Get Current DateTime`**: Captura e formata o timestamp atual (`Topico/time`).
* **`Change: sn/lote`**: Prepara o valor do Serial Number/Lote (`Topico/sn`).
* **MQTT Publish (3x)**: Envia o Tempo, o Lote/SN e a Soma (contagem).

### 2. Contador de Produção (Caminho Central)

Este caminho rastreia e publica a contagem total de produtos.

* **`Change: counter_produtos + 1`**: Incrementa a variável de fluxo (`flow.counter_produtos`) em 1.
* **`Set counter_total_produtos`**: Armazena o valor atualizado.
* **MQTT Publish**: Publica o **total atual** no `Topico/total/produtos`, alimentando o widget **Gauge** (Medidor) no dashboard.

### 3. Controle de Status da Máquina (Caminho Inferior)

Este caminho simula o estado operacional da esteira.

* **`Change: status_maquina = 1`**: Define a variável de status para `1` (Ligada).
* **MQTT Publish**: Publica `1` no `Topico/status/maquina` (acendendo o LED como **Verde**).
* **`Delay for 10 seconds`**: Simula o tempo de operação.
* **`Change: status_maquina = 0`**: Define a variável de status para `0` (Parada).
* **MQTT Publish**: Publica `0` no `Topico/status/maquina` (mudando o LED para **Vermelho**).

---

## 🚀 Como Usar este Fluxo

1.  **Instalação dos Módulos:** Certifique-se de que os módulos **`node-red-dashboard`** e **`node-red-node-ui-led`** (se estiver usando o dashboard clássico) estão instalados na sua paleta do Node-RED.
2.  **Importe o JSON:** Copie o código JSON do seu fluxo e importe-o no Node-RED via **Menu > Importar > Fluxo**.
3.  **Deploy e Acesso:** Clique em **Deploy** e acesse o dashboard em `http://[Node-RED_IP]:1880/ui` para visualizar o LED, o medidor de produtos, o tempo por peça e o gráfico de tendência.
