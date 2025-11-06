# 📡 Configuração do TTN Gateway

Este documento descreve a configuração do gateway **RAK7289V2 (WisGate Edge)** utilizado na rede da **UNICAMP**, conectado ao **The Things Network (TTN)**.  
O objetivo é registrar, configurar e operar o gateway para comunicação LoRaWAN com dispositivos sensores.

---

## 📑 Índice
- [Visão Geral](#-visão-geral)
- [Objetivos](#-objetivos)
- [Conexão Física e Rede](#-conexão-física-e-rede)
- [Acesso ao Painel WisGateOS](#️-acesso-ao-painel-wisgateos)
- [Registro do Gateway no TTN](#-registro-do-gateway-no-ttn)
- [Configuração LoRaWAN no WisGateOS (UDP)](#️-configuração-lorawan-no-wisgateos-udp)
- [Adição e Associação de Dispositivos no TTN](#-adição-e-associação-de-dispositivos-no-ttn)
- [Buffer de Downlink e Limitação Atual](#-buffer-de-downlink-e-limitação-atual)
- [Trabalhos em Andamento: Migração para Basics Station](#-trabalhos-em-andamento-migração-para-basics-station)
- [Resumo Operacional](#-resumo-operacional)

---

## 🔎 Visão Geral
- Gateway: **RAK7289V2 (WisGate Edge)**  
- Conexão: Ethernet → rede cabeada da UNICAMP  
- Função: Receber pacotes **LoRaWAN/LoRa** e encaminhar ao TTN (Community Edition, cluster `nam1`)  
- Registro TTN:  
  - **Gateway ID:** `gateway-unicamp`  
  - **Gateway EUI:** *(único, gravado no hardware)*  
  - **Frequency Plan:** Australia 915-928 MHz, FSB 2 (compatível AU915 Brasil)  
  - **Protocolo:** UDP (Packet Forwarder)  
  - **Status:** ativo, recebendo ~264k uplinks  

> ✅ Uplink funcionando  
> ⚠️ Downlink/ACK em fase de trabalho

---

## 🎯 Objetivos
- Receber mensagens LoRaWAN dos sensores.  
- Encaminhar dados ao TTN em tempo real.  
- Tornar o gateway público no TTN.  
- Permitir múltiplos dispositivos (atualmente 4) publicarem telemetria.  

---

## 🔌 Conexão Física e Rede
1. Ligar o gateway na alimentação externa.  
2. Conectar via cabo Ethernet à rede da UNICAMP.  
3. Liberar porta UDP usada pelo forwarder LoRaWAN.  
4. Confirmar IP via DHCP (usado para acessar o **WisGateOS**).  

---

## 🖥️ Acesso ao Painel WisGateOS
- Acessar endereço local via navegador.  
- Login com credenciais de administrador.  
  - senha: `********`  

**Após login:**  
- Status da interface LoRa  
- Modo de operação (*Packet Forwarder*)  
- Versão do firmware / WisGateOS  
- Temperatura interna  

---

## 🌐 Registro do Gateway no TTN
1. Acessar [TTN Console](https://console.cloud.thethings.network) → cluster `nam1`.  
2. Gateways → + Register gateway.  
3. Preencher:  
   - Gateway ID  
   - Gateway EUI  
   - Frequency plan: AU915 FSB2  
   - Public status: Enabled  
   - Packet Broker forwarding: Enabled  
   - Require authenticated connection: Disabled (modo UDP)  
4. Salvar.  

**Após registro:**  
- “Last activity: just now”  
- Contadores uplink/downlink  
- Métricas (RSSI, SNR, temperatura)  
- Histórico em Live Data  

---

## ⚙️ Configuração LoRaWAN no WisGateOS (UDP)
1. Abrir **LoRa → LoRa Network Settings**.  
2. Selecionar **Packet Forwarder / Semtech UDP**.  
3. Configurar:  
   - Gateway EUI / Station EUI  
   - Server address: cluster NAM1 TTN  
   - Server port  
   - Frequency plan: AU915 FSB2  
4. Salvar e aplicar.  

**Após salvar:**  
- Gateway envia uplinks ao TTN.  
- TTN Console → Live Data exibe mensagens (`Receive uplink message`, RSSI, SNR, Data rate, FCnt).  

---

## 📲 Adição e Associação de Dispositivos no TTN
1. TTN Console → End devices → + Add end device.  
2. Para cada dispositivo:  
   - Device ID (ex.: `bitdog-01`)  
   - DevEUI  
   - JoinEUI/AppEUI  
   - AppKey  
   - Selecionar aplicação TTN correspondente  
3. Verificação:  
   - Após join, TTN exibe em Live Data: `JoinEUI`, `DevEUI`, `FCnt`, `FPort`, Data rate, RSSI, SNR.  

**Importante:**  
- Dispositivos usam **chaves OTAA** (DevEUI, JoinEUI, AppKey).  
- **API Keys TTN** não são usadas nos dispositivos, apenas em integrações externas.  

---

## 📥 Buffer de Downlink e Limitação Atual
- TTN possui opção **Enable server-side buffer of downlink messages**.  
- No modo UDP, gateway não mantém canal seguro para downlinks.  
- Resultado: dispositivos não recebem ACK/confirmations.  

**Impacto:**  
- Dispositivos podem reenviar pacotes sem confirmação.  
- Aumenta tráfego de rádio → observar duty cycle.  

---

## 🔄 Trabalhos em Andamento: Migração para Basics Station
**Motivos:**
- UDP → apenas uplink, sem sessão segura.  
- Basics Station → conexão segura via WebSocket (WSS), habilita downlink/ACK.  

**Passos planejados:**
1. WisGateOS → LoRa Network Settings → trocar para Basics Station.  
2. Configurar:  
   - LNS Server URL  
   - Station EUI  
   - Frequency Plan: AU915 FSB2  
   - API Key (TTN)  
3. Salvar e reiniciar módulo LoRa.  
4. TTN Console → Gateway → Live Data → verificar protocolo Basics Station e eventos `Transmit downlink message`.  

**Após migração:**  
- Downlink/ACK habilitado.  
- Controle de retransmissão.  
- Respeito ao duty cycle obrigatório.  

---

## 📑 Resumo Operacional
- Gateway físico **RAK7289V2** registrado como `gateway-unicamp`.  
- Conectado via Ethernet à rede da UNICAMP, portas UDP liberadas.  
- Frequency plan: **AU915 FSB2**.  
- Recebendo tráfego LoRaWAN de 4 dispositivos registrados no TTN.  
- Dispositivos usam OTAA (DevEUI, JoinEUI/AppEUI, AppKey).  
- Downlink/ACK ainda não ativo.  
- Migração para **Basics Station** em andamento para comunicação bidirecional.  
