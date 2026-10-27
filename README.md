# Lab Windows Server — AD, DNS e DHCP

## Objetivo

Montar do zero um ambiente Windows Server com Active Directory, DNS e DHCP
para consolidar habilidades administrativas (não apenas operacionais) e
servir como base para estudos de SCCM e infraestrutura híbrida.

## Ambiente

- Hyper-V no Windows 11 (host)
- VM: Windows Server 2022 Standard (Desktop Experience)
- Hostname: LAB-DC01
- Rede do lab: 192.168.10.0/24 (switch interna isolada — LAB-NET)
- Rede de internet: Default Switch do Hyper-V

## O que foi configurado

### Active Directory

- Forest/domain novo: lab.local
- Functional level: Windows Server 2016
- DC promovida com DNS e Global Catalog
- DSRM configurada

### DNS (administração manual)

- Zona direta primária: empresa.local
  - Registro A: servidor1 → 192.168.10.20
  - Registro A: teste → 192.168.10.30
  - CNAME: intranet → servidor1.empresa.local
- Zona reversa: 10.168.192.in-addr.arpa
  - PTR: 192.168.10.20 → servidor1.empresa.local
- Forwarders: 8.8.8.8, 1.1.1.1
- Dynamic updates: desabilitado nas zonas manuais (controle explícito dos registros)

### DHCP (administração manual)

- Role instalada e autorizada no AD
- Escopo: Lab-192.168.10 (192.168.10.100–200)
- Exclusão: 192.168.10.180–190
- Reserva: 192.168.10.150 (impressora de teste)
- Opções: gateway 192.168.10.1, DNS 192.168.10.10, domínio lab.local

## Testes e resultados

### DNS

- `nslookup servidor1.empresa.local` → 192.168.10.20 ✅
- `nslookup intranet.empresa.local` → 192.168.10.20 (via CNAME) ✅
- `nslookup 192.168.10.20` → servidor1.empresa.local (via PTR) ✅
- `nslookup google.com` → resolvido via forwarder ✅
- `Get-DnsServerZone` → lab.local, empresa.local, 10.168.192.in-addr.arpa listadas ✅

### DHCP

- `Get-DhcpServerv4Scope` → escopo Lab-192.168.10 ativo ✅
- `Get-DhcpServerv4ExclusionRange` → faixa 192.168.10.180–190 excluída ✅
- `Get-DhcpServerv4Reservation` → reserva 192.168.10.150 criada ✅
- `Get-DhcpServerv4Lease` → cliente pegou IP do escopo ✅

## O que aprendi

- Eu já mexia em AD há mais de dez anos no dia a dia — criava usuário, movia
  computador de OU, resetava senha. Mas montar a floresta do zero me fez
  perceber que eu nunca tinha pensado em *por que* o functional level existe,
  ou *quem* decide o nome do domínio. Operar é seguir o manual; administrar
  é entender por que o manual foi escrito daquele jeito.

- Descobri na prática que DHCP em domínio precisa ser **autorizado no AD**
  antes de distribuir IP. O serviço instala, o escopo aparece bonito na tela,
  mas nada funciona — e o erro não é óbvio. É uma daquelas coisas que só
  fazem sentido quando você vê o "porquê" por trás da tela.

- Zona reversa e PTR pareciam detalhe até eu testar `nslookup 192.168.10.20`
  e ver o nome voltar. Entendi que resolução direta e reversa são duas metades
  da mesma coisa — e que em troubleshooting de rede, log e auditoria, a
  reversa é tão importante quanto a direta.

- Forwarders eu configurei quase no automático, mas o teste `nslookup google.com`
  foi o que me mostrou o motivo real: o DNS interno é autoritativo só para o
  domínio local. Sem forwarder, o servidor não sabe para onde perguntar quando
  o nome é externo. Coisa simples que eu nunca tinha parado para raciocinar.

- Exclusão de escopo DHCP eu já tinha visto em algum lugar, mas agora entendi
  o propósito: reservar faixas para equipamentos com IP fixo — impressoras,
  servidores, switches — para o DHCP nunca entregar aquele endereço por engano.
  É prevenção de conflito de IP, não burocracia.

## Próximos passos

- AZ-900 (Azure Fundamentals) — em andamento
- SCCM: fundamentos, collections, deployments, Task Sequence
- GPO avançada (security baseline, loopback, WMI filter)
- Ambiente híbrido: Entra ID / Azure AD Connect

## Stack

Windows Server 2022 · Active Directory · DNS Server · DHCP Server · Hyper-V
