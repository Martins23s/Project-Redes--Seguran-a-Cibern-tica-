# Configuração Inicial

## Cenário

Empresa com:
- Recepção
- RH
- Financeiro
- TI
- Sala de Servidores

**Equipamentos**
- 1 Switch Multilayer 3650-24PS (Core)
- 3 Switches 2960 (Acesso)
- 12 PCs
- 1 Impressora de rede
- 2 Servidores (DHCP e Arquivos)
- 1 Roteador para Internet

---

**Topologia**
<img width="1103" height="687" alt="image" src="https://github.com/Martins23s/Project-Redes--Seguran-a-Cibern-tica-/commit/65016ae5e5e6aff67d505ed007154596e83635a0" />

---

# VLANs

**VLAN 10 - Recepção**
- Rede: 192.168.10.0/24
- Gateway: 192.168.10.1

**VLAN 20 - RH**
- Rede: 192.168.20.0/24
- Gateway: 192.168.20.1

**VLAN 30 - Financeiro**
- Rede: 192.168.30.0/24
- Gateway: 192.168.30.1

**VLAN 40 - TI**
- Rede: 192.168.40.0/24
- Gateway: 192.168.40.1

**VLAN 50 - Servidores**
- Rede: 192.168.50.0/24
- Gateway: 192.168.50.1

**VLAN 99 - Gerenciamento**
- Rede: 192.168.99.0/24
- Gateway: 192.168.99.1

---

# Distribuição de Portas

**SW1 ( RH e Recepção)**
- Fa0/1-4    VLAN 10 (Recepção)
- Fa0/5-8    VLAN 20 (RH)
- Gi0/1      Trunk para Switch Core

**SW2 ( Financeiro e TI)**
- Fa0/1-4    VLAN 30 (Financeiro)
- Fa0/5-8    VLAN 40 (TI)
- Gi0/1      Trunk para Switch Core

**SW3 (Servidores e Impressora)**
- Gig 6-8/1 – VLAN 50 (Servidores e Impressora)
- Gig 9/1 – Trunk para Switch Core

**CORE-SW**
- **Gig1/0/1** 10,20
- **Gig1/0/2** 10,20,30,40,50,99
- **Gig1/0/3** 20,30,50
- **Gig1/0/24** - 192.168.254.2/30 Ligada ao Router1

**Router 1**
- GI0/1 - 192.168.254.1/30
- GI0/0 - 200.10.10.1/24

---

**Habilitar Roteamento no Switch Core**
`ip routing`

---

## Habilitando Portas Trunk
No **switch core,** configure as portas trunk conforme abaixo:  
- **Gig1/0/1** - SW1 - vlans 10 e 20
- **Gig1/0/2** - SW2 - VLANS 30, 40
- **Gig1/0/3** - SW3 - VLAN 50
```
interface range gi1/0/1-3
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,99
```

**Nos switches de acesso (S1, S2, S3):**
Em cada um dos três switches, selecione a porta conectada do lado do Switch Core e habilite ela para trunk.
 `switchport mode trunk`

---
# Servidores e Impressora
**DHCP**
IP: 192.168.50.10

**Arquivos**
IP: 192.168.50.20

**Impressora**
IP: 192.168.50.30

---
# Regras de Negócio
- RH acessa apenas servidores e internet. ✅
- Financeiro acessa servidores , impressora e internet . ✅
- TI acessa toda a rede e gerencia os equipamentos (Switches, Firewall, Roteador). ✅
	- O TI realiza esse gerenciamento através da VLAN 99
- Recepção não acessa Financeiro. ✅
- Servidores e impressora não tem acesso a Internet. ✅

---

# Configurando Switches
Como você está usando um **Switch Multilayer 3650 (Core)** e **3 switches 2960 de acesso**, as VLANs precisam ser criadas em **todos os switches**, e os links entre eles devem ser configurados como **trunk**. 
## SW1 (Recepção e RH) 
Entre no switch: 
```
enable 
configure terminal 

##Criar as VLANs 

vlan 10 
name RECEPCAO 

vlan 20 
name RH 
```
##### Configurar portas da Recepção 
```
interface range fa0/1-4 
switchport mode access 
switchport access vlan 10 
```
##### Configurar portas do RH 
```
interface range fa0/5-8 
switchport mode access 
switchport access vlan 20 
```

##### Configurar a porta de uplink para o Core 
Supondo que seja a Gi0/1: 
```
interface gi0/1 
switchport mode trunk 
```

Salve: 
```
end 
write 
```

## SW2 (Financeiro, TI) 
```
enable 
configure terminal 
```
##### Criar VLANs 
```
vlan 30 
name FINANCEIRO

vlan 40 
name TI 
```
##### Interfaces - Financeiro 
```
interface range fa0/1-4 
switchport mode access 
switchport access vlan 30 
```
##### Interfaces - TI 
```
interface range fa0/5-8 
switchport mode access 
switchport access vlan 40 
```
##### Link para o Core 
```
interface gi0/1 
switchport mode trunk 
```
Salve: 
```
end 
write 
```

## SW3 (Servidores e Impressora) 
```
enable 
configure terminal 
```

##### Criar VLANs 
```
vlan 20 
name RH 

vlan 30 
name FINANCEIRO

vlan 50
name SERVERS
```

##### Interfaces Servers e Impressora 
```
interface gigabitEthernet 6/1
switchport mode access
switchport access vlan 50

interface gigabitEthernet 7/1
switchport mode access
switchport access vlan 50

interface gigabitEthernet 8/1
switchport mode access
switchport access vlan 50
```
##### Link para o Core 
```
interface gigabitEthernet 9/1
switchport mode trunk

end
write
```

## SW3650 (Core) 
##### Criando as VLANs no Switch Core
```
conf t

vlan 10
 name RECEPCAO

vlan 20
 name RH

vlan 30
 name FINANCEIRO

vlan 40
name TI

vlan 50
name SERVIDORES

vlan 99
name GERENCIA
```

#### Configurar as portas trunk no Switch
Nesse cenário, as interfaces trunk no Switch Core são as seguintes:
- Gi1/0/1 → SW1 
- Gi1/0/2 → SW2 
- Gi1/0/3 → SW3
Essa configuração permite com que as VLANs troquem dados entre si, por isso é necessário configurar as interfaces gi1/0/1 até a gi1/0/3 como Trunk.
```
interface range gi1/0/1-3 
switchport mode trunk 
switchport trunk allowed vlan 10,20,30,40,50,99 
```
#### Configurar os Gateways das VLANs (SVI) 
``` 
interface vlan 10 
ip address 192.168.10.1 255.255.255.0 
no shutdown 

interface vlan 20 
ip address 192.168.20.1 255.255.255.0 
no shutdown 

interface vlan 30 
ip address 192.168.30.1 255.255.255.0 
no shutdown 

interface vlan 40 
ip address 192.168.40.1 255.255.255.0 
no shutdown 

interface vlan 50 
ip address 192.168.50.1 255.255.255.0 
no shutdown 

interface vlan 99 
ip address 192.168.99.1 255.255.255.0 
no shutdown 
```
#### Habilitar roteamento entre VLANs no Switch Core
```
ip routing 
```

## Verificações !
Ver VLANs:
`show vlan brief` 

Ver trunks 
`show interfaces trunk` 

Ver interfaces IP 
`show ip interface brief` 

Ver tabela de rotas:
`show ip route`

## Endereçamento dos PCs 
##### Recepção 
- IP: 192.168.10.x
- Máscara: 255.255.255.0 
- Gateway: 192.168.10.1 
##### RH 
- IP: 192.168.20.x
- Máscara: 255.255.255.0 
- Gateway: 192.168.20.1
##### Financeiro 
- IP: 192.168.30.x
- Máscara: 255.255.255.0
- Gateway: 192.168.30.1 
##### TI 
- IP: 192.168.40.x
- Máscara: 255.255.255.0
- Gateway: 192.168.40.1 
##### Servidor DHCP 
- IP: 192.168.50.1xx
- Máscara: 255.255.255.0 
- Gateway: 192.168.50.1 

Após realizar as configurações no Switch Core, validei as conexões entre as VLANS

---
# Configurando ACL - Access Control List do switch core
**CORE - SW (Switch Multilayer 3650-24PS)** no centro da topologia exercendo a função de fazer o roteamento entre as VLAN's e a Internet. 

- **SW1 (2960)** à esquerda:
	- VLAN RH 
	- VLAN Recepção 

- **SW2 (2960)** à direita: 
	- VLAN TI 
	- VLAN Financeiro 

- **Switch dos servidores** na parte superior: 
	- Servidor DHCP 
	- Servidor Arquivos 
	- Servidor Impressora 

O principal objetivo aqui é:
- RH acessa apenas servidores e internet. ✅
- Financeiro acessa servidores, impressora e internet. ✅
- TI acessa toda a rede. ✅
- Recepção não acessa Financeiro. ✅
- Fazer com que apenas a TI acesse a VLAN 99, pois essa é a VLAN que gerencia os equipamentos ✅

### VLANs 
1     VLAN 10 = Recepção     192.168.10.0/24 
2     VLAN 20 = RH           192.168.20.0/24 
3     VLAN 30 = Financeiro   192.168.30.0/24 
4     VLAN 40 = TI           192.168.40.0/24 
5     VLAN 50 = Servidores   192.168.50.0/24 
6     VLAN 90 = Gerencia 192.168.99.0/24
## Criar ACL para o RH 

A ACL vai: 
✅ Permitir acesso do RH aos servidores e Internet
❌ Bloquear acesso do RH às demais redes 
```
conf t
ip access-list extended RH_ONLY_SERVERS 
	10 permit udp any eq bootpc any eq bootps (8 match(es))
	20 permit udp any eq bootps any eq bootpc
	30 deny ip 192.168.20.0 0.0.0.255 host 192.168.50.30
	40 permit ip 192.168.20.0 0.0.0.255 192.168.50.0 0.0.0.255
	50 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
	60 deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
	70 permit ip 192.168.20.0 0.0.0.255 any
```
As duas primeiras linhas (10 e 20) habilitam o DHCP Client e DHCP server na ACL, permitindo assim que o Switch core funcione como DHCP relay.
## Criar ACL para o Financeiro
✅ Permitir acesso do Financeiro aos servidores e Internet 
❌ Bloquear acesso do Financeiro às demais redes
```
conf t 
ip access-list extended FINANCEIRO
	10 permit udp any eq bootpc any eq bootps (8 match(es))
	20 permit udp any eq bootps any eq bootpc
	30 permit ip 192.168.30.0 0.0.0.255 192.168.50.0 0.0.0.255
	40 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
	50 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
	60 permit ip 192.168.30.0 0.0.0.255 any (6 match(es))
```

As duas primeiras linhas (10 e 20) habilitam o DHCP Client e DHCP server na ACL, permitindo assim que o Switch core funcione como DHCP relay.
## Criar ACL para a Recepcao
✅ Permitir acesso da Recepcao aos servidores, TI e Internet 
❌ Bloquear acesso da Recepcao às demais redes

```
conf t
ip access-list extended Recepcao
	10 permit udp any eq bootpc any eq bootps (10 match(es))
	20 permit udp any eq bootps any eq bootpc
	30 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
	40 permit ip 192.168.10.0 0.0.0.255 any (23 match(es))
```

As duas primeiras linhas (10 e 20) habilitam o DHCP Client e DHCP server na ACL, permitindo assim que o Switch core funcione como DHCP relay.
#### Aplicar na VLAN do RH
```
1     interface vlan 20 
2      ip access-group RH_ONLY_SERVERS in
```
#### Aplicar na VLAN do Financeiro 
```
1     interface vlan 30 
2      ip access-group FINANCEIRO in  
```
#### Aplicar na VLAN da Recepcao 
```
1     interface vlan 10
2      ip access-group Recepcao in  
```

---

# Adicionando o servidor Web Público na topologia 
Adicionei o Servidor WEB topologia, no entanto ainda não conectei ele a nenhum dispositivo.
**Configure:** 
- Servidor WEB Público 
- IP: 200.10.10.10 
- Máscara: 255.255.255.0 
- Gateway: 200.10.10.1 

----------------------------------------------------------------------------------------------------
# Configurando DHCP
Na topologia foi necessário configurar o servidor DHCP de forma manual. Abaixo segue as informações de IP do Servidor DHCP:
- **IP** - 192.168.50.10
- **Subnet Mask** - 255.255.255.0
- **Default gateway** - 192.168.50.1
- **DNS Server** - N/A
- 
## Interface Servidor -> SW3
Na interface a qual o servidor está conectado no switch **SW3**, foi necessário configurar a interface gi7/1 como modo de acesso.
```
enable
configure terminal

interface gi7/1
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast
```

## Interface SW3 - Switch Core:
```
interface gi1/0/3
 switchport mode trunk
 switchport trunk allowed vlan 50
```
No core, a porta correspondente também precisa transportar a VLAN 50, a escolha da interface varia de topologia para topologia; mas nesse caso foi utilizado a porta gi1/0/3, pois é a porta trunk.
## Crie os Pools no Servidor DHCP
1. Habilitei o serviço **DHCP** no servidor de IP 192.168.50.10
2. Criei as seguintes POOLs para cada VLAN:
- VLAN 10 - 192.168.10.2 - 192.168.10.254
- VLAN 20 - 192.168.20.2 - 192.168.20.254
- VLAN 30 - 192.168.30.2 - 192.168.30.254
- VLAN 40 - 192.168.40.2 - 192.168.40.254

## DHCP Relay
Como o servidor DHCP está na VLAN 50, foi necessário configurar o DHCP Relay no switch core, para que fosse possível encaminhar solicitações de endereços IP entre as VLANs e o servidor DHCP. 
1. No Switch Core, configurei o DHCP relay em cada uma das VLANs
```
interface vlan 10
ip helper-address 192.168.50.10

interface vlan 20
ip helper-address 192.168.50.10

interface vlan 30
ip helper-address 192.168.50.10

interface vlan 40
ip helper-address 192.168.50.10

```
## Reconfigurando as ACL's
Para que o servidor **DHCP** alocado na VLAN 50 funcione corretamente nas VLAN's 10,20 e 30, foi necessário alterar a access list das três as VLAN's, ficando da seguinte forma:
```
Extended IP access list FINANCEIRO
	8 permit udp any eq bootpc any eq bootps
	9 permit udp any eq bootps any eq bootpc
	10 permit ip 192.168.30.0 0.0.0.255 192.168.50.0 0.0.0.255
	20 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
	30 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
	40 permit ip 192.168.30.0 0.0.0.255 any
Extended IP access list RH-ACL
	10 permit udp any eq bootpc any eq bootps (3 match(es))
	20 permit udp any eq bootps any eq bootpc
	30 deny ip 192.168.20.0 0.0.0.255 host 192.168.50.30 (1 match(es))
	40 permit ip 192.168.20.0 0.0.0.255 192.168.50.0 0.0.0.255 (2 match(es))
	50 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 (1 match(es))
	60 deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255 (1 match(es))
	70 permit ip 192.168.20.0 0.0.0.255 any (2 match(es))
Extended IP access list Recepcao
	8 permit udp any eq bootpc any eq bootps (2 match(es))
	9 permit udp any eq bootps any eq bootpc
	10 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255 (4 match(es))
	20 permit ip 192.168.10.0 0.0.0.255 any
```
Note que antes de qualquer regra nas ACL's foi necessário adicionar as duas linhas:
```
	8 permit udp any eq bootpc any eq bootps
	9 permit udp any eq bootps any eq bootpc
```
Essas duas linhas permitem o funcionamento do DHCP através da ACL. 
- `bootpc` representa a porta UDP 68, usada pelo **cliente DHCP**
- `bootps` representa a porta UDP 67, usada pelo **Servidor DHCP**.
- A linha `8 permit udp any eq bootpc any eq bootps` ==permite tráfego UDP, vindo de qualquer endereço pela porta de **cliente DHCP (68)**, destinado a qualquer endereço na porta do servidor **DHCP (67)**==.
- Na linha `9 permit udp any eq bootps any eq bootpc`, que é o sentido contrário, ==permite tráfego UDP que venha da porta do **servidor DHCP(67)** e tenha como destino a porta do cliente **DHCP (68)**==.
Resumidamente essas duas linhas permitem o processo do **DHCP**:


---
# Criação de VLAN 99
Essa VLAN será utilizada exclusivamente para gerenciar os switches da infraestrutura:

| Equipamento  | Função                  | IP de gerência |
| ------------ | ----------------------- | -------------- |
| CORE-SW      | Core 3650               | 192.168.99.1   |
| SW1          | Recepção + RH           | 192.168.99.2   |
| SW2          | Financeiro + TI         | 192.168.99.3   |
| SW3          | Servidores e Impressora | 192.168.99.4   |


A única VLAN autorizada a administrar esses equipamentos será a VLAN 40.
**VLAN 40 - TI**
- 192.168.40.0/24

## Criação da VLAN 99 no CORE-SW
No switch core 3650:
```
enable
configure terminal

vlan 99
 name GERENCIA
 
interface vlan 99
 ip address 192.168.99.1 255.255.255.0
 no shutdown
 
 ip-routing
```

## Permitir VLAN 99 nos trunks
Aqui, adicionei a VLAN 99 nas interfaces Trunk do Switch **CORE-SW**.
```
interface range gi1/0/1-3
switchport trunk allowed vlan add 99

#Verifique as configurações com:
show interfaces trunk
```
## Configure os Switches CORE-SW, SW1, SW2 e SW3:
A tabela abaixo mostra o IP de gerência de cada equipamento.

| Equipamento | Função                  | IP de gerência |
| ----------- | ----------------------- | -------------- |
| CORE-SW     | Core 3650               | 192.168.99.1   |
| SW1         | Recepção + RH           | 192.168.99.2   |
| SW2         | Financeiro + TI         | 192.168.99.3   |
| SW3         | Servidores e Impressora | 192.168.99.4   |

1. Criação da VLAN 99 no SW1 e alocação de IP de gerência no switch:
```
enable
configure terminal

vlan 99
 name GERENCIA
 
interface vlan 99
 ip address 192.168.99.2 255.255.255.0
 no shutdown

ip default-gateway 192.168.99.1
```
Fiz essa configurações nos demais switches (SW2 e SW3). Claro, mudando o endereço de IP de acordo com a tabela acima.

## Criar ACL no Switch-Core 3650
Essa ACL servirá para apenas a VLAN 40 (TI) se comunicar com a VLAN 99, que é a VLAN de gerência dos switches.
```
ip access-list extended GERENCIA-ACL
 10 permit ip 192.168.40.0 0.0.0.255 192.168.99.0 0.0.0.255
 20 deny ip any 192.168.99.0 0.0.0.255
 30 permit ip any any
 
#Adicione a ACL na vlan 99>
interface vlan 99
 ip access-group GERENCIA-ACL out
```
A opção deve ser "out" pois o tráfego sairá do CORE-SW em direção aos dispositivos (switches) da vlan 99.

## 1. Adicionando mais proteção com ACL no Switches
Mesmo adicionando a ACL no switch core, ainda não será possível que a **vlan 40 (TI)** gerencie os equipamentos; logo será necessário criar uma ACL para adicionar nas linhas **VTY** dos switches. Em cada um dos equipamentos abaixo, crie uma ACL e adicionei-a nas linhas VTY do equipamento.
- CORE-SW
- SW1
- SW2
- SW3
1. 
```
ip domain-name empresa.local

username admin privilege 15 secret Cisco123

crypto key generate rsa
 1024
 
ip ssh version 2
```
Um pequeno overview do comando acima:
- a primeira linha cria domínio para que seja possível realizar o acesso ao equipamento.
- A segunda linha cria um usuário admin com o privilégio 15 (maior privilégio do equipamento), habilitando o secret, a senha ficará criptografada no switch
- A terceira linha gera um par de chaves criptográficas RSA e ativa o SSH no equipamento. ==Basicamente substitui conexões antigas e inseguras (Telnet) pelo Protocolo SSH.==
- Ativa e força o uso exclusivo do protocolo SSHv2 no equipamento.
Agora, será necessária criar a **ACL para acesso administrativo nos equipamentos:**
2. Nos switches (Switch Core, SW1, SW2 e SW3), realizei as seguintes configurações: 
```
access-list 99 permit 192.168.40.0 0.0.0.255
access-list 99 deny any

line vty 0 15
 login local
 transport input ssh
 access-class 99 in
 
line vty 0 4
 login local
 transport input ssh
```

Após realizar essa configurações será possível realizar o acesso via SSH através de qualquer computador da VLAN 40; utilizando `ssh -l admin <ip do switch>` para realizar acesso em modo privilegiado.

---
# Adicionando Firewall
Adicionei o Firewall ASA 5506-X entre o CORE-SW e o Gateway_Internet, pois o intuíto do firewall nessa topologia é realizar o controle de tráfego **interno -> externo** e vice versa.
## 1. Router - Gateway_Internet
Configuração das portas do roteador:
```
interface g0/0
ip address 200.10.10.1 255.255.255.0
no shutdown

interface g0/1
ip address 192.168.254.5 255.255.255.252
no shutdown
```
-  A interface g0/0 virada para o servidor Web
- A interface g0/1 virada para o firewall ASA
## 2. ASA 5506
Configuração das interfaces do Firewall:
```
interface gigabitEthernet1/1
nameif outside
security-level 0
ip address 192.168.254.6 255.255.255.252
no shutdown

interface gigabitEthernet1/2
nameif inside
security-level 100
ip address 192.168.254.2 255.255.255.252
no shutdown
```
# 3. Core-SW
Conectei a interface 1/2 do ASA na interface gig1/0/24 do CORE-SW e configurei a rota para o IP 192.168.254.2 -- que é a interface GigabitEthernet1/2 do ASA.
```
ip routing

ip route 0.0.0.0 0.0.0.0 192.168.254.2
```
## 4. Rota de retorno - ASA 5506x
Agora é necessário configurar uma **rotas internas de retorno** para cada uma das VLAN's, assim o Firewall ASA saberá para onde encaminhar os pacotes externos.
```
route inside 192.168.10.0 255.255.255.0 192.168.254.1
route inside 192.168.20.0 255.255.255.0 192.168.254.1
route inside 192.168.30.0 255.255.255.0 192.168.254.1
route inside 192.168.40.0 255.255.255.0 192.168.254.1
route inside 192.168.50.0 255.255.255.0 192.168.254.1
route inside 192.168.99.0 255.255.255.0 192.168.254.1
```
Por fim, configurei a rota padrão onde os pacotes irão ser encaminhados para fora da rede, passando pelo **Roteador**.
```
route outside 0.0.0.0 0.0.0.0 192.168.254.5
```
## 5. Criar NAT no Firewall
Por fim, realizei a criação de um NAT pelo Firewall para que os equipamentos da LAN acessem a internet com o IP do Firewall.
```
object network LAN
 subnet 192.168.0.0 255.255.0.0
 nat (inside,outside) dynamic interface
```
## Inspect ICMP e HTTP
Os comando abaixo habilitam o tráfego ICMP e HTTP vindo de interfaces **outside**.
```
policy-map global_policy
 class inspection_default
  inspect icmp
  
policy-map global_policy
 class inspection_default
  inspect http
```

---
# Acesso remoto - Firewall ASA

## 1. Crie usuário local no ASA
```
username admin password Cisco123 privilege 15
```
## 2. Definir um domínio
```
domain-name empresa.local
```
## 3. Gerar as chaves RSA
```
crypkey generat rsa modulus 1024

crypto key generate rsa
```
## 4. Habilitar autenticação local para SSH
```
aaa authentication ssh console LOCAL
```
## 5. Permitir apenas a VLAN 40
Para permitir que os PC's da VLAN 40 se comunicassem com o Firewall:
```
ssh 192.168.40.0 255.255.255.0 inside 
```

## 6. Ajustar o tempo de sessão
Ajustei o tempo de sessão utilizando `ssh timeout 30`. Esse comando basicamente "dropa" sua sessão caso passe por mais 30 minutos sem nenhuma interação.

## 7. Verificando configurações
Verifique as configurações com:
- `show ssh`
- `show run ssh`
Nos pc's da VLAN 40, acessei o firewall com o comando `ssh -l user [IP do equipamento]`

## Security in depth - Core-SW
```
ip access-list extended ASA-MGMT
 permit tcp 192.168.40.0 0.0.0.255 host 192.168.254.2 eq 22
 deny tcp any host 192.168.254.2 eq 22
 permit ip any any
```

Dessa forma, foi adicionada mais uma camada de proteção para acesso via SSH:
- Somente a VLAN 40 consegue na porta SSH do ASA
- A criação da ACL **ASA-MGMT** garante que se alguém conseguir alcançar o IP 192.168.254.2, apenas a VLAN40 poderá iniciar a conexão SSH.

---
# Troubleshooting
Obtive um problema para acessar o modo privilegiado dos switches, então realizei a seguinte configuração em cada um dos Switches:
```
conf t

username admin privilege 15 secret Cisco123

line vty 0 4
 login local
 transport input ssh

end
wr
```
