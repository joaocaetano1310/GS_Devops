# 🌱 AgroVision — Detecção de Pragas via Satélite

> Challenge 2026 — 1º Semestre | FIAP | DevOps Tools & Cloud Computing

---

## 👥 Equipe

| Nome | RM |
|------|-----|
| João Victor Caetano Alves da Silva | RM 562074 |
| João Victor Bueno Castelini da Silva | RM 564115 |
| Ryan Vetoriano | RM 565667 |
| Felipe Furlanetto | RM 562766 |
| Raul Rezende Iemini Aguiar | RM 564002 |

---

## 🔗 Links

- **GitHub:** https://github.com/SEU_REPOSITORIO_AQUI
- **Vídeo YouTube: https://youtu.be/tDT_qkTjMUc

---

## 📋 Descrição da Solução

O **AgroVision** é uma plataforma inteligente voltada para produtores rurais que utiliza análise de imagens de satélite para identificar plantações com pragas. A solução permite que agricultores monitorem suas lavouras de forma remota e eficiente, recebendo alertas sobre áreas afetadas e tomando decisões rápidas para proteger sua produção.

A API foi desenvolvida em **Java Spring Boot** com banco de dados **Oracle**, containerizada com **Docker** e implantada em uma **VM Linux na Azure**.

### 🎯 Benefícios para o Negócio

- **Monitoramento remoto:** produtores acompanham lavouras sem precisar ir a campo
- **Detecção precoce de pragas:** identificação via satélite reduz perdas na produção
- **Histórico centralizado:** registro completo de plantações, safras e insumos em um único lugar
- **Escalabilidade em nuvem:** infraestrutura Azure com Docker garante disponibilidade e fácil expansão
- **Decisões mais rápidas:** relatórios automatizados auxiliam na tomada de decisão

---

## 🏗️ Arquitetura Macro

```
┌─────────────────────────────────────────────────────────────┐
│                  Azure Cloud (Canada Central)                │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │           Resource Group: rg-AgroVision                │  │
│  │                                                        │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │       VM Ubuntu 22.04 (Standard_D2s_v3)          │  │  │
│  │  │                                                  │  │  │
│  │  │  ┌──────────────┐      ┌───────────────────┐    │  │  │
│  │  │  │ app-562074   │─────▶│    db-562074       │    │  │  │
│  │  │  │ Spring Boot  │      │  Oracle Free       │    │  │  │
│  │  │  │ porta 8080   │      │  porta 1521        │    │  │  │
│  │  │  └──────────────┘      └───────────────────┘    │  │  │
│  │  │         │                        │               │  │  │
│  │  │         └────── rede-agrovision ─┘               │  │  │
│  │  │                  volume-oracle-562074             │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
         ▲
         │  HTTP :8080
         │
    👤 Usuário / App Mobile
```

---

## 🛠️ Tecnologias Utilizadas

- Java 21 + Spring Boot 3.3
- Oracle Database (`gvenzl/oracle-free`)
- Docker
- Microsoft Azure (VM Ubuntu 22.04)
- Azure CLI

---

## 🚀 How To — Passo a Passo

### Pré-requisitos
- Azure CLI instalado e logado (`az login`)
- Git instalado na sua máquina

---

### Parte 1 — Provisionando a VM na Azure

Execute o script de criação da VM:

```bash
chmod +x GS_Devops.sh
sed -i 's/\r$//' GS_Devops.sh
./GS_Devops.sh
```

Após a execução, pegue o IP público da VM:

```bash
az vm show \
  --resource-group rg-AgroVision \
  --name vm-AgroVision \
  --show-details \
  --query publicIps
```

Conecte na VM via SSH:

```bash
ssh azureuser@<IP_PUBLICO>
```

---

### Parte 2 — Dentro da VM

**1. Clonar o repositório:**
```bash
git clone https://github.com/SEU_REPOSITORIO_AQUI.git
cd agrovision-gs
```

**2. Criar a rede Docker:**
```bash
docker network create rede-agrovision
```

**3. Criar o volume para o Oracle:**
```bash
docker volume create volume-oracle-562074
```

**4. Subir o container do banco Oracle:**
```bash
docker run -d \
  --name db-562074 \
  --network rede-agrovision \
  -v volume-oracle-562074:/opt/oracle/oradata \
  -e ORACLE_PASSWORD=oracle \
  gvenzl/oracle-free
```

**5. Aguardar o Oracle inicializar** (pode demorar 2-3 minutos):
```bash
docker logs -f db-562074
```
Espere aparecer `DATABASE IS READY TO USE!` e pressione `Ctrl+C`.

**6. Build da imagem da aplicação:**
```bash
docker build -t app-562074 .
```

**7. Subir o container da aplicação:**
```bash
docker run -d \
  --name app-562074 \
  --network rede-agrovision \
  -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=docker \
  app-562074
```

**8. Verificar os logs da aplicação:**
```bash
docker logs -f app-562074
```

**9. Verificar usuário não-privilegiado:**
```bash
docker exec -it app-562074 whoami
docker exec -it app-562074 pwd
docker exec -it app-562074 ls -l
```

---

### Parte 3 — Validar o banco de dados

```bash
docker exec -it db-562074 sqlplus system/oracle@FREEPDB1
```

Dentro do SQLPlus:
```sql
SELECT * FROM TB_USER_GS;
SELECT * FROM TB_PLANTACOES_GS;
SELECT * FROM TB_SAFRA_GS;
SELECT * FROM TB_INSUMO_GS;
SELECT * FROM TB_RELATORIO_GS;
EXIT;
```

---

### Parte 4 — Acessar a aplicação

Substitua `<IP_PUBLICO>` pelo IP da sua VM:

- **API:**     `http://<IP_PUBLICO>:8080`
- **Swagger:** `http://<IP_PUBLICO>:8080/swagger-ui/index.html`

---

### Parte 5 — Deletar a VM ao final

```bash
az group delete --name rg-AgroVision --yes --no-wait
```

---

## 📡 Endpoints da API

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/api/auth/login` | Autenticação (gera token JWT) |
| POST | `/api/usuarios` | Cadastrar usuário |
| GET | `/api/usuarios` | Listar usuários |
| GET | `/api/usuarios/{id}` | Buscar usuário por ID |
| PUT | `/api/usuarios/{id}` | Atualizar usuário |
| DELETE | `/api/usuarios/{id}` | Deletar usuário |
| POST | `/api/plantacoes` | Cadastrar plantação |
| GET | `/api/plantacoes` | Listar plantações |
| GET | `/api/plantacoes/{id}` | Buscar plantação por ID |
| PATCH | `/api/plantacoes/{id}/status` | Atualizar status da plantação |
| DELETE | `/api/plantacoes/{id}` | Deletar plantação |
| POST | `/api/safras` | Registrar safra |
| GET | `/api/safras` | Listar safras |
| GET | `/api/safras/plantacao/{id}/total` | Total colhido por plantação |
| POST | `/api/insumos` | Cadastrar insumo |
| GET | `/api/insumos` | Listar insumos |
| POST | `/api/relatorios` | Gerar relatório |
| GET | `/api/relatorios` | Listar relatórios |

---

## 🔧 Troubleshooting

```bash
# Ver containers rodando
docker ps

# Logs da aplicação
docker logs -f app-562074

# Logs do banco
docker logs -f db-562074

# Remover containers
docker rm -f app-562074
docker rm -f db-562074

# Remover volume (apaga os dados!)
docker volume rm volume-oracle-562074
```


> FIAP — 2026 | Turma 2TDS | DevOps Tools & Cloud Computing
