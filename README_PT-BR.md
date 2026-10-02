# ⚡ KIX Deployment Suite: Camada de Pagamento Autônoma

![Bitcoin](https://img.shields.io/badge/Bitcoin-Lightning-orange)
![Docker](https://img.shields.io/badge/Docker-Obrigatório-blue)
![Licenção](https://img.shields.io/badge/Licen%C3%A7a-MIT-green)
![Arquitetura](https://img.shields.io/badge/Foco-Autonomia-black)

O KIX é um **Framework Independente para Comerciantes** projetado para integrar negócios à Lightning Network do Bitcoin. Desenvolvido como resposta direta a arquiteturas de pagamento centralizadas e modelos automatizados de rastreamento fiscal (como os mecanismos de *split payment* automatizado que o Brasil planeja adotar), o KIX replica a experiência de usuário simplificada dos pagamentos instantâneos via QR Code, preservando a independência operacional financeira.

🌐 **Informações gerais e documentação:**
<https://satoshicanvas.com/kix-eng/>

---

## 🇧🇷 A Estratégia "Pix": Familiaridade e Adoção

O Brasil treinou com sucesso 150 milhões de pessoas para adotar transações instantâneas via QR Code. O KIX aproveita esse hábito consolidado para reduzir a fricção na adoção de pagamentos alternativos.

- **Fluxo Familiar:** O KIX mantém uma experiência de usuário idêntica à dos trilhos de pagamento instantâneo padrão, evitando barreiras complexas de onboarding.
- **Autonomia Operacional:** Ao utilizar a Lightning Network como camada de liquidação, comerciantes operam de forma independente de intermediários bancários tradicionais e de eventuais restrições de conta.
- **Retenção de Valor:** Projetado para otimizar as margens do comerciante, eliminando as altas taxas de intermediários associadas às processadoras convencionais de crédito e débito.

---

## 🏗️ Arquitetura do Sistema

```text
                 ┌────────────────────────────┐
                 │     Interface Comerciante  │
                 │   (KIX Dashboard-homepage) │
                 └─────────┬──────────────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │      LNbits       │
                 │  Motor de Pagto   │
                 │   PRISMA-4/5      │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │   Nó Lightning    │
                 │ Phoenixd / Alby   │
                 │    PRISMA-4       │
                 └─────────┬─────────┘
                           │
                ┌──────────┴──────────┐
                │                     │
                ▼                     ▼
        Serviço Oculto Tor      Gateway VPS
          PRISMA-2               PRISMA-2
          (acesso .onion)        (Porta 80)
```

O sistema utiliza encapsulamento Docker para fornecer escalabilidade horizontal em um único host. Cada instância contém:

- **Dashboard:** Uma interface web centralizada (Homepage) para gerenciamento.
- **Engine (LNbits):** Um poderoso processador de pagamentos da Lightning Network. **(PRISMA-4 / PRISMA-5)**
- **Vault (Phoenixd / Alby Hub):** Gerenciamento seguro de chaves e conectividade leve com o nó. **(PRISMA-4)**
- **Tor Bridge:** Um contêiner de roteamento baseado em Alpine que gera URLs Onion únicas em tempo real. **(PRISMA-2)**

O ambiente host subjacente — como um servidor Debian Linux x86_64 — pertence ao **PRISMA-1**.

A fonte de cadeia do Bitcoin e o mecanismo de sincronização pertencem ao **PRISMA-3**. Esta camada é implementada de acordo com o backend Lightning utilizado. Ao usar o **Alby Hub**, o backend gerencia sua própria conectividade e sincronização de cadeia conforme sua arquitetura, enquanto o Nostr Wallet Connect (NWC) fornece a interface de conectividade da carteira usada pelas camadas superiores. Ao usar o **Phoenixd**, a infraestrutura Bitcoin/Lightning e a sincronização são gerenciadas pela **arquitetura ACINQ/Phoenix**.

---

## 🚀 Primeiros Passos

### 1. Requisitos

- **Host Debian / Linux x86_64** **(PRISMA-1)**
- **Docker** **(PRISMA-1)**
- **Docker Compose** **(PRISMA-1)**
- **Privilégios sudo** (necessários para gerenciamento de volumes e leitura de hostnames do Tor)
- **Auxiliar de entropia** (opcional, mas recomendado para Tor) **(PRISMA-2)**

Se o Tor estiver lento para gerar chaves:

```bash
sudo apt install haveged
```

> O Docker deve estar instalado e em execução antes de executar qualquer script de implantação.

### 2. Métodos de Instalação e Implantação

| # | Método | Script | Descrição |
|---|--------|--------|-----------|
| 1 | **Autônomo** | `kix_tor_sovereign.sh` | 🌑 Implantação avançada de privacidade via Tor **(PRISMA-2)**. Executa o LNbits com Alby Hub **(PRISMA-4)** e utiliza sua arquitetura de conectividade/sincronização de carteira **(PRISMA-3)**. Requer configurar a chave Nostr do AlbyHub dentro do LNbits. |
| 2 | **Phoenixd** | `kix_phoenixd.sh` | 🔥 Nó Phoenix multi-instância ultraleve **(PRISMA-4)** + LNbits **(PRISMA-4/5)** + Tor **(PRISMA-2)**. A sincronização de cadeia é tratada pela arquitetura Phoenix/ACINQ **(PRISMA-3)**. |
| 3 | **VPS Clearnet** | `kix_vps_dedicated.sh` | 🌍 Implantação pública em clearnet **(PRISMA-2)** em uma VPS dedicada **(PRISMA-1)** (Porta 80). |

---

### 2.1 O Método Phoenixd Multi-Hunter (Recomendado)

Este método utiliza o orquestrador `kix_phoenixd.sh` para implantar nós Phoenix isolados. Inclui gerenciamento automático de volumes e backups de ambiente.

O host de implantação faz parte do **PRISMA-1**, enquanto o Phoenixd fornece o motor Lightning no **PRISMA-4**.

#### Configuração

```bash
sudo chmod +x kix_phoenixd.sh
```

#### Implantar Instância (Padrão é v1)

```bash
./kix_phoenixd.sh v1
```

#### Implantar Instância "v8"

```bash
./kix_phoenixd.sh v8
```

#### Como funciona

**Isolamento**

Cria um diretório como:

```text
~/PHOENIX_[ID]
```

Cada instância possui volumes Docker únicos, evitando colisões. Esta infraestrutura persistente pertence ao **PRISMA-1**.

**Configuração Automática do LNbits**

O script `kix_phoenixd.sh` configura automaticamente o LNbits com o endpoint e a senha corretos da API do Phoenixd.

O backend Lightning é o **PRISMA-4**, enquanto a interface de faturas/pagamentos fornecida pelo LNbits opera no **PRISMA-5**.

Após a implantação:

- O LNbits já está conectado ao nó Phoenix
- A carteira está pronta para gerar faturas Lightning imediatamente
- Nenhuma configuração manual de API é necessária

> A geração de faturas Lightning faz parte do **PRISMA-5**.

#### PRISMA-3 — Sincronização de Cadeia Phoenix / ACINQ

Com o Phoenixd, a camada de fonte de cadeia e sincronização é tratada pela infraestrutura Phoenix/ACINQ.

A implantação KIX não precisa operar uma instância separada do Bitcoin Core para a configuração Phoenixd descrita aqui. O Phoenix gerencia a sincronização Bitcoin/Lightning necessária conforme sua própria arquitetura.

Portanto:

```text
PRISMA-1 → Debian / Linux x86_64 + Docker
PRISMA-2 → Tor / exposição de rede
PRISMA-3 → Sincronização Phoenix / ACINQ
PRISMA-4 → Motor Lightning Phoenixd
PRISMA-5 → LNbits / geração de faturas Lightning
```

#### Importante

O Phoenix abre automaticamente seu primeiro canal Lightning quando a carteira recebe fundos.

> ⚠️ Os primeiros ~20.000 sats são usados pela ACINQ para abrir o canal inicial.

Por esse motivo, é crítico armazenar com segurança as palavras-semente (seed) do Phoenix. A seed é a única forma de recuperar a carteira e seus fundos caso o nó ou servidor seja perdido.

#### Credenciais

O script gera:

- uma senha de API de 16 hex
- arquivo `.env`
- `env_backup.txt`

#### Descoberta

O script varre os volumes Docker para localizar:

```text
seed.dat
```

e imprime o endereço Tor `.onion` gerado.

> O endereço `.onion` gerado pertence ao **PRISMA-2**.

---

### 2.2 O Método Autônomo (Tor + Alby Hub)

Esta é a configuração de máxima autonomia. Ela executa:

- LNbits **(PRISMA-4 / PRISMA-5)**
- Serviço oculto Tor **(PRISMA-2)**
- Backend de carteira Alby Hub **(PRISMA-4)**

#### PRISMA-3 — Fonte de Cadeia e Sincronização do Alby Hub

Quando o KIX usa o Alby Hub, o PRISMA-3 é fornecido pela arquitetura de carteira/backend utilizada pelo Alby Hub.

A implantação KIX não mantém necessariamente sua própria instância do Bitcoin Core. O Alby Hub gerencia sua própria carteira subjacente e conectividade/sincronização com a blockchain conforme sua arquitetura.

O Nostr Wallet Connect (NWC) fornece a interface de conectividade da carteira entre aplicações como o LNbits e a carteira Alby Hub.

Portanto:

```text
PRISMA-1 → Debian / Linux x86_64 + Docker
PRISMA-2 → Serviço Oculto Tor
PRISMA-3 → Alby Hub / sua arquitetura de sincronização de cadeia
PRISMA-4 → Alby Hub + backend Lightning do LNbits
PRISMA-5 → LNbits / geração de faturas
```

> O LNbits deve ser conectado ao Alby Hub usando a chave Nostr.

#### Configuração do AlbyHub

1. Abra o Alby Hub
2. Copie sua chave privada Nostr
3. Cole-a nas configurações do LNbits
4. Isso vincula o LNbits à carteira Alby Hub.

O backend Alby Hub / Lightning é o **PRISMA-4**.

A interface de pagamento do LNbits e a geração de faturas operam no **PRISMA-5**.

A sincronização de blockchain subjacente do Alby Hub pertence ao **PRISMA-3**.

#### Executar

```bash
sudo chmod +x kix_tor_sovereign.sh
./kix_tor_sovereign.sh 1
```

#### Acesso

Use o Tor Browser e abra a URL `.onion` impressa ao final do script.

> O endpoint `.onion` é o **PRISMA-2**.

Aguarde 1–2 minutos para a propagação do circuito Tor.

---

### 2.2.1 O Método Multi-Instância com Bind Mount

Esta é a evolução do Método Autônomo, projetada especificamente para alta disponibilidade e backups fáceis. Diferente dos volumes Docker padrão, este método usa **Bind Mounts**, mapeando os dados do contêiner diretamente para pastas visíveis no seu diretório home.

O host físico e o armazenamento persistente pertencem ao **PRISMA-1**.

#### Vantagens Principais

- **Visibilidade de Dados:** Todos os bancos de dados Lightning e chaves Tor ficam armazenados em `~/KIX_PROTOTIPO[ID]/data`.
- **Backups Fáceis:** Você pode fazer backup de todo o seu nó simplesmente copiando uma pasta local, sem comandos complexos de exportação de volumes.
- **Robustez:** Evita perda de dados durante atualizações do Docker ou migrações de contêineres.

#### Implantação

O número no final é o número da instância:

```bash
sudo chmod +x kix_multi_tor_bind_mount.sh
./kix_multi_tor_bind_mount.sh 9
```

#### Estrutura de Pastas Criada

O script organiza automaticamente seus diretórios operacionais:

- `~/KIX_PROTOTIPO9/data/lnbits` — Bancos de dados SQLite e extensões. **(PRISMA-4 / PRISMA-5)**
- `~/KIX_PROTOTIPO9/data/alby` — Suas chaves e configurações do Alby Hub. **(PRISMA-4)**
- `~/KIX_PROTOTIPO9/data/tor` — Endereços `.onion` permanentes (não mudam se o contêiner reiniciar). **(PRISMA-2)**

#### Pós-Implantação

Depois que o script imprimir seus links `.onion`, lembre-se de:

1. Acessar seu Alby Hub pelo link Tor. **(PRISMA-2 → PRISMA-4)**
2. Vinculá-lo ao LNbits usando o Nostr Wallet Connect (NWC) ou a chave interna da conta. **(PRISMA-4)**
3. O Alby Hub gerencia sua conectividade e sincronização de blockchain subjacentes. **(PRISMA-3)**

Seus dados agora são persistentes e fisicamente localizados em `~/KIX_PROTOTIPO9`.

---

### 2.3 Método VPS Dedicada (Clearnet)

Use isto quando a VPS for dedicada exclusivamente ao KIX e você quiser desempenho máximo na Porta 80.

A VPS e seu ambiente Linux/Docker pertencem ao **PRISMA-1**.

A exposição pública em Clearnet na Porta 80 pertence ao **PRISMA-2**.

O backend Lightning pertence ao **PRISMA-4**, enquanto sua geração de faturas e codificação de pagamento pertencem ao **PRISMA-5**.

O mecanismo de fonte de cadeia/sincronização usado pelo backend Lightning selecionado pertence ao **PRISMA-3**.

#### Executar

```bash
sudo chmod +x kix_vps_dedicated.sh
./kix_vps_dedicated.sh 1
```

---

## 🔮 Integrações Futuras e Nota de Conformidade

À medida que as arquiteturas estatais evoluem para sistemas automatizados de desvio fiscal multipartite (como a divisão automática obrigatória de impostos no ponto de venda), o KIX serve como uma **linha de base não-split-payment** (modelo de referência "nusplit"), garantindo autocustódia nativa.

Iterações futuras podem incluir hooks de exportação opcionais e plugins modulares projetados para interfacear de forma limpa com padrões governamentais de nota fiscal eletrônica (como APIs de *Nota Fiscal* / NFE), permitindo que comerciantes mantenham conformidade fiscal transparente enquanto preservam o controle local sobre a liquidez de liquidação.

---

## 🛠️ Operações

### Monitorar Recursos dos Contêineres

Ver rede e I/O de disco em tempo real:

```bash
sudo docker stats
```

### Limpar Dados Docker Órfãos

Remover volumes Docker não utilizados de experimentos antigos:

```bash
sudo docker volume prune -f
```

---

## 🔐 Aviso de Segurança

O KIX é um framework de código aberto para gestão financeira e independência de infraestrutura.

Os usuários são responsáveis por:

- Suas chaves e backups
- Sua postura de segurança do nó
- Sua conformidade regulatória e fiscal local

---

## 📦 Metadados do Repositório

**Repositório:** `kix-protocol-suite`

### Tópicos

```text
bitcoin
lightning-network
lnbits
phoenixd
albyhub
pix-brazil
self-custody
privacy
autonomy
prisma
prisma-1
prisma-2
prisma-3
prisma-4
prisma-5
```
