# Projeto  - Linux - Infraestrutura Web com Monitoramento Automatizado

Este projeto configura um servidor **Nginx** no **Ubuntu/Linux Mint**, adiciona uma página HTML simples, implementa um script de monitoramento que verifica a disponibilidade do site e envia notificações via **Webhook no Discord**, além de configurar o `systemd` para garantir que o Nginx reinicie automaticamente em caso de falha.

---

##  Sumário
1. [Instalação e Configuração do Nginx](#-instalação-e-configuração-do-nginx)
2. [Criação da Página HTML](#-criação-da-página-html)
3. [Script de Monitoramento com Webhook Discord](#-script-de-monitoramento-com-webhook-discord)
4. [Agendamento com Cron](#-agendamento-com-cron)
5. [Configuração de Reinício Automático no Systemd](#-configuração-de-reinício-automático-no-systemd)
6. [Estrutura de Arquivos](#-estrutura-de-arquivos)
7. [Autor](#-autor)

---

## Instalação e Configuração do Nginx

Atualize o sistema e instale o Nginx:
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install nginx -y
```
Verifique o status do Nginx:
```bash
sudo systemctl status nginx
```
---
## Criação da Página HTML
Crie o diretório do site: 
```bash
sudo mkdir -p /var/www/meusite
```
Crie o arquivo HTML:
```bash
sudo nano /var/www/meusite/index.html
```
Preencha com o conteúdo:
```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Site da empresa</title>
</head>
<body>
    <h1>Esse é o site da empresa, sejam bem vindos!</h1>
    <p>Desculpa! Estamos em manutenção no momento...</p>
</body>
</html>
```
Configure o site no Nginx:
```bash
sudo nano /etc/nginx/sites-available/meusite
```
Preencha com o conteúdo:
```nginx
server {
    listen 80;
    server_name localhost;

    root /var/www/meusite;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Ative a configuração:
```bash
sudo ln -s /etc/nginx/sites-available/meusite /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```
---

## Script de Monitoramento com Webhook Discord

#### *Criação do Webhook Discord*
Vá até o canal do Discord onde você quer receber as notificações.
Clique no nome do canal → Editar Canal → Integrações → Webhooks.
Clique em "Novo Webhook", dê um nome e copie a URL do Webhook (vai começar com https://discord.com/api/webhooks/...).

Crie o script
```bash
sudo nano /usr/local/bin/verifica_site.sh
```
Preencha com o conteúdo:
```bash
#!/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

SITE_URL="http://localhost"
LOG_FILE="/var/log/meu_script.log"
DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/SEU WEBHOOK AQUI"

TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" "$SITE_URL")

log() {
    echo "[$TIMESTAMP] $1" >> "$LOG_FILE"
}

enviar_discord() {
    local mensagem="$1"
    curl -s -H "Content-Type: application/json" \
         -X POST \
         -d "{\"content\": \"$mensagem\"}" \
         "$DISCORD_WEBHOOK_URL" > /dev/null
}

if [ "$HTTP_CODE" -eq 200 ]; then
    log "Site está online. Código HTTP: $HTTP_CODE"
else
    MSG=" *Alerta!*: O site $SITE_URL está fora do ar! Código HTTP: $HTTP_CODE - $TIMESTAMP"
    log "$MSG"
    enviar_discord "$MSG"
fi
```
Dê permissão e crie o log:
```bash
sudo chmod +x /usr/local/bin/verifica_site.sh
sudo touch /var/log/meu_script.log
sudo chmod 664 /var/log/meu_script.log
```
<img width="1279" height="332" alt="image" src="https://github.com/user-attachments/assets/52890c24-e6f8-4a58-99a8-84d22b2551ec" />

---

## Agendamento com cron
Edite o crontab
```bash
crontab -e
```
Adicione ao fim da página o conteúdo:
```bash
* * * * * /usr/local/bin/verifica_site.sh
```
Verifique se está funcionando:
```bash
crontab -l
```
<img width="1187" height="633" alt="image2" src="https://github.com/user-attachments/assets/5418a373-8c87-4f66-b313-3475e1dffbfc" />
<img width="1313" height="67" alt="image3" src="https://github.com/user-attachments/assets/37a252df-633d-4cff-a78a-890ccfc7db3f" />

---

## Configuração de Reínicio Automático no systemd
Edite a configuração do serviço:
```bash
sudo systemctl edit nginx
```
Preencha com o conteúdo:
```ini
[Service]
Restart=always
RestartSec=5
```
Recarregue e reinicie para aplicar alterações:
```bash
sudo systemctl daemon-reexec
sudo systemctl restart nginx
```
<img width="1116" height="216" alt="image1" src="https://github.com/user-attachments/assets/1fe75551-d60e-4718-91e5-e36e8d0a371b" />

---
## Estrutura de Arquivos
```bash
/var/www/meusite/index.html           # Página HTML
/etc/nginx/sites-available/meusite    # Configuração do site no Nginx
/usr/local/bin/verifica_site.sh       # Script de monitoramento
/var/log/meu_script.log               # Log de monitoramento
```
---

## Autor
### *Guilherme Assumpção Lourenzo*
