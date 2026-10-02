# ⚡ KIX Deployment Suite: Autonomous Payment Layer

![Bitcoin](https://img.shields.io/badge/Bitcoin-Lightning-orange)
![Docker](https://img.shields.io/badge/Docker-Required-blue)
![License](https://img.shields.io/badge/License-MIT-green)
![Architecture](https://img.shields.io/badge/Focus-Autonomy-black)

KIX is an **Independent Merchant Framework** designed to integrate businesses with the Bitcoin Lightning Network. Engineered as a direct response to centralized payment architectures and automated fiscal tracing models (such as Brazil's upcoming automated *split payment* mechanisms), KIX replicates the streamlined user experience of instant QR-code payments while preserving financial operational independence.

🌐 **General information and documentation:**
<https://satoshicanvas.com/kix-eng/>

---

## 🇧🇷 The "Pix" Strategy: Familiarity and Adoption

Brazil successfully trained 150 million people to adopt instant QR-code transactions. KIX leverages this established habit to reduce friction in alternative payment adoption.

- **Familiar Flow:** KIX maintains a user experience identical to standard instant payment rails, avoiding complex onboarding barriers.
- **Operational Autonomy:** By utilizing the Lightning Network as a settlement layer, merchants operate independently from traditional banking intermediaries and potential account restrictions.
- **Value Retention:** Designed to optimize merchant margins by eliminating high intermediary fees associated with conventional credit and debit processors.

---

## 🏗️ System Architecture

```text
                 ┌────────────────────────────┐
                 │        Merchant UI         │
                 │  (KIX Dashboard-homepage)  │
                 └─────────┬──────────────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │      LNbits       │
                 │  Payment Engine   │
                 │   PRISMA-4/5      │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │  Lightning Node   │
                 │ Phoenixd / Alby   │
                 │    PRISMA-4       │
                 └─────────┬─────────┘
                           │
                ┌──────────┴──────────┐
                │                     │
                ▼                     ▼
        Tor Hidden Service     VPS Gateway
          PRISMA-2               PRISMA-2
          (.onion access)         (Port 80)
```

The system utilizes Docker encapsulation to provide horizontal scalability on a single host. Each instance contains:

- **Dashboard:** A centralized web interface (Homepage) for management.
- **Engine (LNbits):** A powerful Lightning Network payment processor. **(PRISMA-4 / PRISMA-5)**
- **Vault (Phoenixd / Alby Hub):** Secure key management and lightweight node connectivity. **(PRISMA-4)**
- **Tor Bridge:** An Alpine-based routing container that generates unique Onion URLs in real-time. **(PRISMA-2)**

The underlying host environment — such as a Debian Linux x86_64 server — belongs to **PRISMA-1**.

The Bitcoin chain source and synchronization mechanism belong to **PRISMA-3**. This layer is implemented according to the Lightning backend being used. When using **Alby Hub**, the backend handles its chain connectivity and synchronization according to its own architecture, while Nostr Wallet Connect (NWC) provides the wallet connectivity interface used by the upper layers. When using **Phoenixd**, the Bitcoin/Lightning infrastructure and synchronization are managed by the **ACINQ/Phoenix architecture**.

---

## 🚀 Getting Started

### 1. Requirements

- **Debian / Linux x86_64 host** **(PRISMA-1)**
- **Docker** **(PRISMA-1)**
- **Docker Compose** **(PRISMA-1)**
- **Sudo privileges** (required for volume management and reading Tor hostnames)
- **Entropy helper** (optional but recommended for Tor) **(PRISMA-2)**

If Tor is slow generating keys:

```bash
sudo apt install haveged
```

> Docker must be installed and running before executing any deployment scripts.

### 2. Installation & Deployment Methods

| # | Method | Script | Description |
|---|--------|--------|-------------|
| 1 | **Autonomous** | `kix_tor_sovereign.sh` | 🌑 Advanced privacy deployment via Tor **(PRISMA-2)**. Runs LNbits with Alby Hub **(PRISMA-4)** and uses its wallet connectivity/synchronization architecture **(PRISMA-3)**. Requires configuring the Nostr key from AlbyHub inside LNbits. |
| 2 | **Phoenixd** | `kix_phoenixd.sh` | 🔥 Ultra-lightweight multi-instance Phoenix node **(PRISMA-4)** + LNbits **(PRISMA-4/5)** + Tor **(PRISMA-2)**. Chain synchronization is handled by the Phoenix/ACINQ architecture **(PRISMA-3)**. |
| 3 | **Clearnet VPS** | `kix_vps_dedicated.sh` | 🌍 Public clearnet deployment **(PRISMA-2)** on a dedicated VPS **(PRISMA-1)** (Port 80). |

---

### 2.1 The Phoenixd Multi-Hunter Method (Recommended)

This method uses the `kix_phoenixd.sh` orchestrator to deploy isolated Phoenix nodes. It includes automatic volume management and environment backups.

The deployment host is part of **PRISMA-1**, while Phoenixd provides the Lightning engine at **PRISMA-4**.

#### Setup

```bash
sudo chmod +x kix_phoenixd.sh
```

#### Deploy Instance (Default is v1)

```bash
./kix_phoenixd.sh v1
```

#### Deploy Instance "v8"

```bash
./kix_phoenixd.sh v8
```

#### How it works

**Isolation**

Creates a directory like:

```text
~/PHOENIX_[ID]
```

Each instance has unique Docker volumes preventing collisions. This persistent infrastructure belongs to **PRISMA-1**.

**Automatic LNbits Configuration**

The `kix_phoenixd.sh` script automatically configures LNbits with the correct Phoenixd API endpoint and password.

The Lightning backend is **PRISMA-4**, while the invoice/payment interface provided by LNbits operates at **PRISMA-5**.

After deployment:

- LNbits is already connected to the Phoenix node
- The wallet is ready to generate Lightning invoices immediately
- No manual API configuration is required

> Lightning invoice generation is part of **PRISMA-5**.

#### PRISMA-3 — Phoenix / ACINQ Chain Synchronization

With Phoenixd, the chain source and synchronization layer is handled by the Phoenix/ACINQ infrastructure.

The KIX deployment does not need to operate a separate Bitcoin Core instance for the Phoenixd configuration described here. Phoenix manages the required Bitcoin/Lightning synchronization according to its own architecture.

Therefore:

```text
PRISMA-1 → Debian / Linux x86_64 + Docker
PRISMA-2 → Tor / network exposure
PRISMA-3 → Phoenix / ACINQ synchronization
PRISMA-4 → Phoenixd Lightning engine
PRISMA-5 → LNbits / Lightning invoice generation
```

#### Important

Phoenix automatically opens its first Lightning channel when the wallet receives funds.

> ⚠️ The first ~20,000 sats are used by ACINQ to open the initial channel.

For this reason it is critical to safely store the Phoenix seed words. The seed is the only way to recover the wallet and its funds if the node or server is lost.

#### Credentials

The script generates:

- a 16-hex API password
- `.env` file
- `env_backup.txt`

#### Discovery

The script scans Docker volumes to locate:

```text
seed.dat
```

and prints the generated Tor `.onion` address.

> The generated `.onion` address belongs to **PRISMA-2**.

---

### 2.2 The Autonomous Method (Tor + Alby Hub)

This is the maximum autonomy configuration. It runs:

- LNbits **(PRISMA-4 / PRISMA-5)**
- Tor hidden service **(PRISMA-2)**
- Alby Hub wallet backend **(PRISMA-4)**

#### PRISMA-3 — Alby Hub Chain Source & Synchronization

When KIX uses Alby Hub, PRISMA-3 is provided by the wallet/backend architecture used by Alby Hub.

The KIX deployment does not necessarily maintain its own Bitcoin Core instance. Alby Hub manages its own underlying wallet and blockchain connectivity/synchronization according to its architecture.

Nostr Wallet Connect (NWC) provides the wallet connectivity interface between applications such as LNbits and the Alby Hub wallet.

Therefore:

```text
PRISMA-1 → Debian / Linux x86_64 + Docker
PRISMA-2 → Tor Hidden Service
PRISMA-3 → Alby Hub / its chain synchronization architecture
PRISMA-4 → Alby Hub + LNbits Lightning backend
PRISMA-5 → LNbits / invoice generation
```

> LNbits must be connected to Alby Hub using the Nostr key.

#### AlbyHub Configuration

1. Open Alby Hub
2. Copy your Nostr private key
3. Paste it inside LNbits settings
4. This links LNbits to the Alby Hub wallet.

The Alby Hub / Lightning backend is **PRISMA-4**.

The LNbits payment interface and invoice generation operate at **PRISMA-5**.

The underlying Alby Hub blockchain synchronization belongs to **PRISMA-3**.

#### Run

```bash
sudo chmod +x kix_tor_sovereign.sh
./kix_tor_sovereign.sh 1
```

#### Access

Use Tor Browser and open the `.onion` URL printed at the end of the script.

> The `.onion` endpoint is **PRISMA-2**.

Allow 1–2 minutes for Tor circuit propagation.

---

### 2.2.1 The Multi-Instance Bind Mount Method

This is the evolution of the Autonomous Method, specifically engineered for high-availability and easy backups. Unlike standard Docker volumes, this method uses **Bind Mounts**, mapping the container data directly to visible folders in your home directory.

The physical host and persistent storage belong to **PRISMA-1**.

#### Key Advantages

- **Data Visibility:** All Lightning database and Tor keys are stored in `~/KIX_PROTOTIPO[ID]/data`.
- **Easy Backups:** You can backup your entire node by simply copying a local folder, without complex volume export commands.
- **Robustness:** Prevents data loss during Docker updates or container migrations.

#### Deployment

The number at the end is the instance number:

```bash
sudo chmod +x kix_multi_tor_bind_mount.sh
./kix_multi_tor_bind_mount.sh 9
```

#### Folder Structure Created

The script automatically organizes your operational directories:

- `~/KIX_PROTOTIPO9/data/lnbits` — SQLite databases and extensions. **(PRISMA-4 / PRISMA-5)**
- `~/KIX_PROTOTIPO9/data/alby` — Your Alby Hub keys and settings. **(PRISMA-4)**
- `~/KIX_PROTOTIPO9/data/tor` — Permanent `.onion` addresses (they won't change if the container restarts). **(PRISMA-2)**

#### Post-Deployment

After the script prints your `.onion` links, remember to:

1. Access your Alby Hub via the Tor link. **(PRISMA-2 → PRISMA-4)**
2. Link it to LNbits using the Nostr Wallet Connect (NWC) or the account's internal key. **(PRISMA-4)**
3. Alby Hub manages its underlying blockchain connectivity and synchronization. **(PRISMA-3)**

Your data is now persistent and physically located at `~/KIX_PROTOTIPO9`.

---

### 2.3 Dedicated VPS Method (Clearnet)

Use this when the VPS is dedicated exclusively to KIX and you want maximum performance on Port 80.

The VPS and its Linux/Docker environment belong to **PRISMA-1**.

The public Clearnet exposure on Port 80 belongs to **PRISMA-2**.

The Lightning backend belongs to **PRISMA-4**, while its invoice generation and payment encoding belong to **PRISMA-5**.

The chain source/synchronization mechanism used by the selected Lightning backend belongs to **PRISMA-3**.

#### Run

```bash
sudo chmod +x kix_vps_dedicated.sh
./kix_vps_dedicated.sh 1
```

---

## 🔮 Future Integrations & Compliance Note

As state architectures evolve toward automated multi-party fiscal diversion systems (such as mandatory automated tax splitting at the point of sale), KIX serves as a **non-split-payment baseline** ("nusplit" reference model) ensuring native self-custody.

Future iterations may include optional export hooks and modular plugins designed to interface cleanly with government electronic invoice standards (such as *Nota Fiscal* / NFE APIs), allowing merchants to maintain transparent fiscal compliance while preserving local control over settlement liquidity.

---

## 🛠️ Operations

### Monitor Container Resources

View real-time network and disk I/O:

```bash
sudo docker stats
```

### Cleanup Dangling Docker Data

Remove unused Docker volumes from old experiments:

```bash
sudo docker volume prune -f
```

---

## 🔐 Security Disclaimer

KIX is an open-source framework for financial management and infrastructure independence.

Users are responsible for:

- Their keys and backups
- Their node security posture
- Their local regulatory and fiscal compliance

---

## 📦 Repository Metadata

**Repository:** `kix-protocol-suite`

### Topics

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
