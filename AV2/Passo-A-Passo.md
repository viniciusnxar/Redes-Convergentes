## 1. Planejamento de endereçamento

| Rede        | Máscara         | Gateway em Roteador |
| ----------- | --------------- | ------------------- |
| **VLAN 10** | 10.0.10.0/24    | 10.0.10.1           |
| **VLAN 20** | 10.0.20.0/24    | 10.0.20.1           |
| **VLAN 30** | 10.0.30.0/24    | 10.0.30.1           |
| **R0–R1**   | 192.168.12.0/30 | R0: .1 R1: .2       |
| **R1–R2**   | 192.168.23.0/30 | R1: .1 R2: .2       |

- PC1 em VLAN 10, PC2 em VLAN 20, PC3 e PC4 em VLAN 30.

---

## 2. Configuração dos switches de acesso

### SW0

1. Crie as VLANs 10 e 20 e 30

```shell
Switch> enable
Switch# configure terminal
  vlan 10
    name VLAN10-Administrativo
  exit
  vlan 20
    name VLAN20-Vendas
  exit
  vlan 30
    name VLAN30-TI
  exit
end
Switch# write memory

```

2. Configure as portas de tronco para acesso ao SW1 e R0

```shell
Switch# configure terminal
  interface Fa0/1
    switchport mode trunk
  exit
  interface Fa0/2
    switchport mode trunk

  exit
end
Switch# write memory
```

3. Configure a porta de uplink ao roteador (Fa0/24) como trunk

```shell
Switch# configure terminal
  interface Fa0/24
    switchport mode trunk
  exit
end
Switch# write memory

```

---

### SW1 ( PC1 e PC2)

```shell
Switch> enable
Switch# configure terminal

!--- Criar VLANs
Switch(config)# vlan 10
Switch(config-vlan)# name VLAN10-Administrativo
Switch(config-vlan)# exit

Switch(config)# vlan 20
Switch(config-vlan)# name VLAN20-Vendas
Switch(config-vlan)# exit

Switch(config)# vlan 30
Switch(config-vlan)# name VLAN30-TI
Switch(config-vlan)# exit

!--- Atribuir portas de usuário
Switch(config)# interface range Fa0/1
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10
Switch(config-if-range)# exit

Switch(config)# interface range Fa0/2
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 20
Switch(config-if-range)# exit

!--- Uplink para o switch de distribuição
Switch(config)# interface Fa0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# exit

Switch(config)# end
Switch# write memory
```

---

### SW 2

1. Crie as VLANs 10 e 20 e 30

```shell
Switch> enable
Switch# configure terminal
  vlan 10
    name VLAN10-Administrativo
  exit
  vlan 20
    name VLAN20-Vendas
  exit
  vlan 30
    name VLAN30-TI
  exit
end
Switch# write memory

```

2. Configure as portas de tronco para acesso ao SW1 e R0

```shell
Switch# configure terminal
  interface Fa0/1
    switchport mode trunk
  exit
  interface Fa0/2
    switchport mode trunk

  exit
end
Switch# write memory
```

---

### SW3 (PC3 e PC4)

```shell
Switch> enable
Switch# configure terminal

!--- Criar VLANs
Switch(config)# vlan 10
Switch(config-vlan)# name VLAN10-Administrativo
Switch(config-vlan)# exit

Switch(config)# vlan 20
Switch(config-vlan)# name VLAN20-Vendas
Switch(config-vlan)# exit

Switch(config)# vlan 30
Switch(config-vlan)# name VLAN30-TI
Switch(config-vlan)# exit

!--- Atribuir PC3 e PC4
Switch(config)# interface range Fa0/1-2
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 30
Switch(config-if-range)# exit

!--- Uplink para o switch de distribuição
Switch(config)# interface Fa0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# exit

Switch(config)# end
Switch# write memory
```

---

## 3. Configuração dos roteadores

### Roteador R0 (atende VLAN 10 e VLAN 20 e link a R1)

```shell
Router0> enable
Router0# configure terminal

!--– Ativando a interface Gi0/0/0
Router0(config)# interface GigabitEthernet0/0/0
Router0(config-if)# no shutdown

!--– Sub‑interfaces para VLANs na Gi0/0/0
Router0(config)# interface GigabitEthernet0/0/0.10
Router0(config-subif)# encapsulation dot1Q 10
Router0(config-subif)# ip address 10.0.10.1 255.255.255.0
Router0(config-subif)# exit

Router0(config)# interface GigabitEthernet0/0/0.20
Router0(config-subif)# encapsulation dot1Q 20
Router0(config-subif)# ip address 10.0.20.1 255.255.255.0
Router0(config-subif)# exit

!--– Link para R1 agora em Gi0/0/1
Router0(config)# interface GigabitEthernet0/0/1
Router0(config-if)# no shutdown
Router0(config-if)# ip address 192.168.12.1 255.255.255.252
Router0(config-if)# exit

!--– Roteamento RIP v2
Router0(config)# router rip
Router0(config-router)# version 2
Router0(config-router)# no auto-summary
Router0(config-router)# network 10.0.10.0
Router0(config-router)# network 10.0.20.0
Router0(config-router)# network 192.168.12.0
Router0(config-router)# exit

Router0(config)# end
Router0# write memory
```

### Roteador central R1 (somente transit)

```shell
Router1> enable
Router1# configure terminal

!--– Interface para R0 em Gi0/0/0
Router1(config)# interface GigabitEthernet0/0/0
Router1(config-if)# no shutdown
Router1(config-if)# ip address 192.168.12.2 255.255.255.252
Router1(config-if)# exit

!--– Interface para R2 em Gi0/0/1
Router1(config)# interface GigabitEthernet0/0/1
Router1(config-if)# no shutdown
Router1(config-if)# ip address 192.168.23.1 255.255.255.252
Router1(config-if)# exit

!--– Roteamento RIP v2
Router1(config)# router rip
Router1(config-router)# version 2
Router1(config-router)# no auto-summary
Router1(config-router)# network 192.168.12.0
Router1(config-router)# network 192.168.23.0
Router1(config-router)# exit

Router1(config)# end
Router1# write memory
```

### Roteador R2 (atende VLAN 30 e link a R1)

```shell
Router2> enable
Router2# configure terminal

!--– Ativando a interface Gi0/0/0
Router0(config)# interface GigabitEthernet0/0/0
Router0(config-if)# no shutdown

!--– Sub‑interface para VLAN 30 em Gi0/0/0
Router2(config)# interface GigabitEthernet0/0/0.30
Router2(config-subif)# encapsulation dot1Q 30
Router2(config-subif)# ip address 10.0.30.1 255.255.255.0
Router2(config-subif)# exit

!--– Link para R1 em Gi0/0/1
Router2(config)# interface GigabitEthernet0/0/1
Router2(config-if)# no shutdown
Router2(config-if)# ip address 192.168.23.2 255.255.255.252
Router2(config-if)# exit

!--– Roteamento RIP v2
Router2(config)# router rip
Router2(config-router)# version 2
Router2(config-router)# no auto-summary
Router2(config-router)# network 10.0.30.0
Router2(config-router)# network 192.168.23.0
Router2(config-router)# exit

Router2(config)# end
Router2# write memory
```

---

## 4. Configuração dos PCs

Em cada PC, defina IP e gateway correspondentes:

- **PC1** (VLAN 10):
  IP: 10.0.10.10
  Máscara: 255.255.255.0
  Gateway: 10.0.10.1

- **PC2** (VLAN 20):
  IP: 10.0.20.20
  Máscara: 255.255.255.0
  Gateway: 10.0.20.1

- **PC3** (VLAN 30):
  IP: 10.0.30.30
  Máscara: 255.255.255.0
  Gateway: 10.0.30.1

- **PC4** (VLAN 30):
  IP: 10.0.30.40
  Máscara: 255.255.255.0
  Gateway: 10.0.30.1

---

## 5. Testando a conectividade

1. **Do PC1**

   ```bash
   ping 10.0.20.20   !– deve chegar em PC2
   ping 10.0.30.30   !– deve chegar em PC3
   ```

2. **Do PC3**

   ```bash
   ping 10.0.10.10   !– deve chegar em PC1
   ping 10.0.20.20   !– deve chegar em PC2
   ```

3. **Verificando tabelas RIP nos roteadores**

   ```shell
   RouterX# show ip route rip
   ```

---
