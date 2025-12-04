---
marp: true
theme: default
paginate: true
---

# Simulação de Rede com Docker Compose
**Trabalho de Redes de Computadores**

---

## Arquitetura da Rede

```
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│  host_one   │◄───────►│   router    │◄───────►│  host_two   │
│192.168.10.10│         │  Gateway    │         │192.168.20.30│
└─────────────┘         └─────────────┘         └─────────────┘
      net_one          192.168.10.254              net_two
   (192.168.10.0/24)   192.168.20.254         (192.168.20.0/24)
```

**3 containers** conectados por **2 redes isoladas**

---

## Container: Router

**Função:** Gateway entre as duas redes

**Configurações principais:**
- Interface eth0 em `net_one` (192.168.10.254)
- Interface eth1 em `net_two` (192.168.20.254)
- IP forwarding habilitado (`net.ipv4.ip_forward=1`)

**Regras iptables:**
- `MASQUERADE` para NAT na eth0
- `FORWARD` permite tráfego entre eth1 ↔ eth0

---

## Container: host_one

**Rede:** `net_one` (192.168.10.0/24)
**IP:** 192.168.10.10

**Configuração:**
- Gateway padrão: 192.168.10.254 (router)
- Ferramentas instaladas: `iproute2`, `iputils-ping`, `net-tools`

**Comando:** Remove rotas antigas e adiciona o router como gateway

---

## Container: host_two

**Rede:** `net_two` (192.168.20.0/24)
**IP:** 192.168.20.30

**Configuração:**
- Gateway padrão: 192.168.20.254 (router)
- Ferramentas instaladas: `iproute2`, `iputils-ping`, `net-tools`

**Comando:** Remove rotas antigas e adiciona o router como gateway

---

## Redes Docker

### O que é Driver Bridge?
- Cria uma **rede virtual privada** no host Docker
- Containers na mesma bridge podem se comunicar diretamente
- Isolamento: bridges diferentes = redes isoladas
- Análogo a um **switch virtual** conectando containers

---

### net_one
- Driver: `bridge`
- Subnet: **192.168.10.0/24**
- Conecta: `host_one` e `router`

### net_two
- Driver: `bridge`
- Subnet: **192.168.20.0/24**
- Conecta: `host_two` e `router`

---

## Fluxo de Comunicação

### host_one → host_two

1. **host_one** (192.168.10.10) envia pacote para 192.168.20.30
2. Pacote vai para o **gateway** (192.168.10.254)
3. **Router** recebe na eth0 (net_one)
4. **iptables** faz FORWARD do pacote
5. **Router** envia pela eth1 (net_two)
6. Pacote chega ao **host_two** (192.168.20.30)

---

## Recursos Especiais

### Capabilities (`cap_add: NET_ADMIN`)
- Permite manipulação de rotas e interfaces de rede
- Necessário para `iptables` e `ip route`

### Sysctls
- `net.ipv4.ip_forward=1` no router
- Habilita encaminhamento de pacotes entre interfaces

### Comando `tail -f /dev/null`
- Mantém os containers em execução
- Permite acesso via `docker exec`

---

## Comandos Úteis

**Iniciar a infraestrutura:**
```bash
docker-compose up -d
```

**Testar conectividade (de host_one para host_two):**
```bash
docker exec -it trab.redes.v-host_one-1 ping 192.168.20.30
```

**Acessar o router:**
```bash
docker exec -it trab.redes.v-router-1 bash
```