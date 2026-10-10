# Projeto de Redes - Gerenciamento de VLAN's; Access-Lists e controle de Tráfego pelo Firewall.
Este projeto representa uma infraestrutura de rede corporativa segmentada por VLANs, utilizando um switch camada 3 para roteamento interno, firewall ASA para controle de acesso à Internet e serviços centralizados em uma VLAN dedicada.
Para informações detalhadas do processo de configuração dos equipamentos, consulte o arquivo NetworkConfig.md no repositório.

## Regras de Negócio:
- O CORE-SW atua como DHCP Relay para distribuir os IP's para as VLANs 10,20,30 e 40.
- RH (VLAN 20) acessa apenas Servidores e Internet
- Financeiro (VLAN 30) acessa servidores, impressora e Internet
- TI (VLAN 40) acessa toda a rede e gerencia os equipamentos de rede (Switches e Firewall)
- A Recepção (VLAN 10) tem acesso à toda rede e à Internet, mas não acessa o financeiro (VLAN50)
- Servidores e Impressora não tem acesso à Internet

## Objetivos da Arquitetura 
Abaixo, citei algumas características dessa arquitetura para facilitar a interpretação da rede corporativa, sem necessidade de verificar a configuração de todos os equipamentos.
- Essa arquitetura é voltada especificamente para trabalhar conceitos de ACLs; roteamento entre VLANs; e configuração básica de Firewall;
- Dividindo a rede em VLANs, possibilita a redução de domínio de broadcast;
- Centralização de serviços;
- Proteção do acesso à Internet atráves do Firewall;
- Possibilidade de expansão da rede;

## Topologia Lógica
<img width="1103" height="687" alt="image" src="https://github.com/Martins23s/Project-Redes--Seguran-a-Cibern-tica-/blob/65016ae5e5e6aff67d505ed007154596e83635a0/ssq.png"/>



---
# Documentação dos equipamentos

## Endereçamento IP e VLANs

| VLAN | Departamento | Rede | Gateway |
|--------|--------|--------|--------|
| 10 | Recepção | 192.168.10.0/24 | 192.168.10.1 |
| 20 | RH | 192.168.20.0/24 | 192.168.20.1 |
| 30 | Financeiro | 192.168.30.0/24 | 192.168.30.1 |
| 40 | TI | 192.168.40.0/24 | 192.168.40.1 |
| 50 | Servidores | 192.168.50.0/24 | 192.168.50.1 |
| 99 | Gerência | 192.168.99.0/24 | 192.168.99.1 |
| Externa | Internet | 200.10.10.0/24 | 200.10.10.1 |
| Trânsito | Core ↔ ASA ↔ Gateway | 192.168.254.0/30 e 192.168.254.4/30 | N/A |

---

## Equipamentos

| Equipamento | Função | IP |
|------------|---------|---------|
| CORE-SW | Switch Core Layer 3 | 192.168.99.1 |
| SW1 | Access Switch | 192.168.99.2 |
| SW2 | Access Switch | 192.168.99.3 |
| SW3 | Access Switch | 192.168.99.4 |
| ASA 5506-X | Firewall | 192.168.254.2 |
| Gateway_Internet | Roteador de borda | 192.168.254.5 |
| Servidor Web Público | Serviço HTTP | 200.10.10.10 |

---

## Servidores e Dispositivos

| Equipamento | Endereço IP |
|------------|---------|
| DHCP Server | 192.168.50.10 |
| Arquivos Server | 192.168.50.20 |
| Printer0 | 192.168.50.30 |
| Servidor Web Público | 200.10.10.10 |

---

## CORE-SW

### Interfaces VLAN (SVI)

| Interface VLAN | Endereço |
|---------------|----------|
| VLAN 10 | 192.168.10.1/24 |
| VLAN 20 | 192.168.20.1/24 |
| VLAN 30 | 192.168.30.1/24 |
| VLAN 40 | 192.168.40.1/24 |
| VLAN 50 | 192.168.50.1/24 |
| VLAN 99 | 192.168.99.1/24 |

Além da configuração das interfaces no Switch Core, foi necessário criar Access List's para cumprir os requisitos das regras de negócio e habilitar a configuração DHCP vinda do Servidor DHCP dentre as VLANs. 

**Access-List "FINANCEIRO":**
- Essas regras permitem com que o Financeiro se comunique com qualquer dispositivo da VLAN 50 e à Internet, mas não se comunique com a VLAN 10 e 20

**Access-List "RH-ACL"**
- Linhas 30 e 40 permitem com que o RH se comunique com com qualquer dispositivo da VLAN 50, menos a Impressora (192.168.50.30)
- As linhas 50 e 60 não permitem a comunicação com dispositivos da VLAN 10 nem VLAN 30.
- A linha 70 permite com que os PC's da VLAN 20 (RH) se comuniquem com a Internet.

**Access-List "Recepcao"**
- A linha 30 bloqueia comunicação da Recepção com o Financeiro
- Linha 40 permite com que a Recepção (VLAN 10) acesse qualquer outro dispositivo ou VLAN

**Access-List "GERENCIA"**
- Essa ACL's permite que apenas os dispositivos da VLAN 40 (TI) acessem dispostivos da VLAN 99.

**Access-List "99"**
- ACL criada justamente para garantir que apenas os PC's da VLAN 40 acessassem o modo administrativos dos equipamentos (CORE-SW, SW1, SW2 e SW3). Antes de criar essa VLAN

**Access-List "ASA-GERENCIA"**
- Access List criada para permitir o gerenciamento via SSH do Firewall somente através de dispositivos da VLAN 40.

Vale ressaltar que nas ACL's FINANCEIRO, RH-ACL e Recepcao, as duas primeiras linhas representam o seguinte:
- A linha `8 permit udp any eq bootpc any eq bootps` permite tráfego UDP, vindo de qualquer endereço pela porta de **cliente DHCP (68)**, destinado a qualquer endereço na porta do servidor **DHCP (67)**.
- Na linha `9 permit udp any eq bootps any eq bootpc`, que é o sentido contrário, ==permite tráfego UDP que venha da porta do **servidor DHCP(67)** e tenha como destino a porta do cliente **DHCP (68)**.

### Links Trunk

| Interface | Destino | VLANs |
|------------|----------|----------|
| Gi1/0/1 | SW1 | 10,20,99 |
| Gi1/0/2 | SW2 | 30,40,99 |
| Gi1/0/3 | SW3 | 50,99 |
| Gi1/0/24 | ASA Inside | Trânsito para Internet |

---

## SW1

| Interface | VLAN | Dispositivo |
|------------|----------|----------|
| Fa0/1-4 | 10 | PCs Recepção |
| Fa0/5-8 | 20 | PCs RH |
| Gi0/1 | Trunk | CORE-SW |

---

## SW2

| Interface | VLAN | Dispositivo |
|------------|----------|----------|
| Fa0/1-4 | 30 | PCs Financeiro |
| Fa0/5-8 | 40 | PCs TI |
| Gi0/1 | Trunk | CORE-SW |

A única ACL existente no equipamento é a "99" que permite gerenciamento no modo privilegiado através apenas da VLAN 40.

---

## SW3

| Interface | VLAN | Dispositivo |
|------------|----------|----------|
| Gi6-7/1 | 50 | Servidores |
| Gi8/1 | 50 | Impressora |
| Gi0/1 | Trunk | CORE-SW |

A única ACL existente no equipamento é a "99" que permite gerenciamento no modo privilegiado através apenas da VLAN 40.

---

# Firewall ASA 5506-X

## Interfaces

| Interface | Nome | Endereço |
|------------|---------|---------|
| Gi1/1 | outside | 192.168.254.6/30 |
| Gi1/2 | inside | 192.168.254.2/30 |

## Funções
- NAT para acesso à Internet.
- Controle de tráfego entre rede interna e externa.
- Gerenciamento remoto via SSH através da VLAN 40 (TI).
- Inspeção de tráfego ICMP e TCP.

---

# Gateway de Internet

| Interface | Endereço |
|------------|---------|
| Gi0/0 | 200.10.10.1/24 |
| Gi0/1 | 192.168.254.5/30 |

---

# Políticas de Segurança

## VLAN de Gerência
A VLAN 99 é utilizada exclusivamente para gerenciamento dos dispositivos de rede:
- CORE-SW
- SW1
- SW2
- SW3

## Controle de Acesso
Para cumprir os objetivos abaixo, foi necessário realizar a criação e gerenciamento de ACL's (Access-Lists) nos equipamentos da topologia.
- Somente a VLAN 40 (TI) pode acessar os dispositivos da VLAN 99.
- Somente a VLAN 40 pode gerenciar o firewall ASA.
- Usuários das demais VLANs não possuem acesso administrativo aos dispositivos.
- Todas as VLANs se comunicam com a Internet, exceto a VLAN 50 (Servidores e impressora)
- Recepção (VLAN 20) não se comunica com Financeiro (VLAN 30)

## Firewall
- Todo o tráfego para Internet passa obrigatoriamente pelo ASA.
- O ASA realiza NAT das redes internas.
- O tráfego de retorno é controlado por inspeção stateful.

---

# Rotas Principais

## Core-SW
Encaminha o tráfego para a interface Gigabit1/2 do Firewall ASA 5506-X.
```cisco
ip route 0.0.0.0 0.0.0.0 192.168.254.2
```

## ASA
O ASA encaminha o tráfego para a interface Gigabit0/1 do Gateway_Internet
```cisco
route outside 0.0.0.0 0.0.0.0 192.168.254.5
```

---

# Serviços Disponíveis

| Serviço | Endereço |
|----------|----------|
| HTTP | 200.10.10.10 |
| DHCP | 192.168.50.10 |
| Arquivos | 192.168.50.20 |
| Impressão | 192.168.50.30 |

---

## Lições aprendidas 
✅ VLANs  
✅ Trunking  
✅ Inter-VLAN Routing  
✅ Switch Multilayer 3650  
✅ DHCP  
✅ Servidores  
✅ Segurança com ACLs  
✅ Planejamento de rede corporativa
