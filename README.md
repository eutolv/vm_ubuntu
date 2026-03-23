````markdown
# 🖥️ VM Ubuntu – Infra/Sysadmin Playground

Este repositório documenta a criação e configuração completa de uma **VM Ubuntu 24.04 LTS** feita do zero para estudo e demonstração de habilidades em administração de sistemas, segurança e servidores web.

A VM é **headless**, com **SSH seguro**, **firewall**, **fail2ban** e **Nginx**.

---

## 1️⃣ Criação e Configuração Inicial

Criar a VM no **Oracle VirtualBox** com as seguintes especificações:

- 🖥️ **Sistema:** Ubuntu 24.04.4 LTS (Noble Numbat)  
- ⚙️ **CPU:** 2 cores  
- 💾 **RAM:** 2 GiB  
- 🗄️ **Disco:** 25 GiB  

Instalar o **Ubuntu Server** do zero:

- ⌨️ Configurar layout de teclado, timezone e idioma  
- 👤 Criar usuários:
  - `tolv` (sem SSH)
  - `herlitz` (com SSH)  
- 🔑 Configurar senha do root se necessário  
- 🛠️ Configurar GRUB (modo seguro usado inicialmente para redefinir senha)  

---

## 2️⃣ SSH – Acesso Seguro

### 2.1 🔐 Configuração da chave

Criar **chave RSA 4096 bits** para o usuário `herlitz`:

```bash
ssh-keygen -t rsa -b 4096 -C "herlitz@vm"
````

Copiar a chave pública para `~/.ssh/authorized_keys` do usuário `herlitz`.

### 2.2 ⚙️ Configuração do SSH no servidor

* Verificar porta SSH (padrão 22)
* Desabilitar login por senha:

```bash
sudo nano /etc/ssh/sshd_config
# Alterar ou adicionar:
PasswordAuthentication no
```

* Reiniciar SSH:

```bash
sudo systemctl restart ssh
```

* Mensagem de boas-vindas personalizada:

```bash
echo "Bem-vindo ao servidor Herlitz - Ubuntu" | sudo tee /etc/motd
```

### 2.3 💻 Configuração do cliente (WSL)

Criar `~/.ssh/config`:

```text
Host herlitz-vm
    HostName 192.168.15.71
    User herlitz
    IdentityFile ~/.ssh/id_rsa
```

Testar conexão:

```bash
ssh herlitz-vm
```

> ✅ Entrou direto com chave, sem pedir senha.

---

## 3️⃣ 🛡️ Firewall (UFW)

Instalar e verificar status:

```bash
sudo apt install ufw -y
sudo ufw status
```

Permitir SSH e ativar:

```bash
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw status verbose
```

Configuração de segurança básica:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

---

## 4️⃣ 🚨 Fail2Ban – Proteção SSH

Instalar:

```bash
sudo apt install fail2ban -y
```

Criar configuração local:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Editar `[sshd]`:

```ini
[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 3600
```

Reiniciar e verificar status:

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd
```

---

## 5️⃣ 🌐 Nginx – Servidor Web

### 5.1 ⚙️ Instalação

```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

### 5.2 🔍 Teste básico

* Acessar no host: `http://192.168.15.71/`
* Ou via terminal:

```bash
curl http://192.168.15.71/
```

### 5.3 🏗️ Configuração de site simples

```bash
sudo mkdir -p /var/www/meusite
sudo chown -R herlitz:herlitz /var/www/meusite
echo '<h1>Olá do Herlitz VM!</h1>' > /var/www/meusite/index.html
```

Criar configuração do site:

```bash
sudo nano /etc/nginx/sites-available/meusite
```

Adicionar:

```nginx
server {
    listen 80;
    server_name 192.168.15.71;

    root /var/www/meusite;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Habilitar site e reiniciar Nginx:

```bash
sudo ln -s /etc/nginx/sites-available/meusite /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Liberar HTTP no firewall:

```bash
sudo ufw allow 'Nginx HTTP'
```

---

## 6️⃣ 🖼️ Troubleshooting VBox – Recursos Gráficos

* Tentativa de habilitar copiar/colar entre host e VM com **VBoxGuestAdditions**: ❌ falhou

  > Motivo: VM sem interface gráfica
* Modo escalonado habilitado para terminal → ajuste manual de fonte/resolução
* Observação: apenas teste, sem impacto funcional na VM

---

## 7️⃣ 📄 Relatório do Sistema (`report.txt`)

* **Kernel:** 6.8.0-71-generic, x86_64
* **CPU:** AMD Ryzen 5 3500U, dual-core 2.096 MHz
* **RAM:** 2 GiB (24% usado)
* **Discos:** `/dev/sda` 25 GiB, `/` 11.23 GiB, `/boot` 1.9 GiB, swap 2 GiB
* **Rede:** enp0s3 1 Gbps full duplex, IPv4 192.168.15.71
* **Gráficos:** Headless, driver `vmwgfx`, OpenGL llvmpipe
* **Áudio:** Intel AC97
* **Processos ativos:** 115
* **Pacotes instalados:** 1031
* **Sensores:** indisponíveis

---

## 8️⃣ 📝 Observações Finais

* ✅ VM simula um **servidor real**: headless, SSH seguro, firewall ativo, fail2ban e Nginx configurados
* ✅ Todas as configurações feitas **manualmente**, sem ISO pré-configurada
* ✅ Documentação pronta para **portfólio de infra/sysadmin**
* ✅ Inclui troubleshooting, configuração de SSH, firewall, fail2ban e servidor web

```
```
