# ConfigurarSSH
# Guia Completo – Configuração de SSH com Chave (ED25519)

Este guia segue exatamente a linha de raciocínio solicitada, com comandos claros e separados entre CLIENTE (seu PC) e SERVIDOR.

## 1. Fazer login no servidor (ainda com senha)

**No CLIENTE (seu PC)**
  ssh usuario@IP_DO_SERVIDOR


Se o login funcionar, continue.

## 2. Criar a chave SSH no cliente

**No CLIENTE**
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_servidor -C "ghostty-servidor"


Pressione Enter para não usar passphrase (login automático).

Arquivos criados:
- `~/.ssh/id_ed25519_servidor`
- `~/.ssh/id_ed25519_servidor.pub`

## 3. Enviar a chave pública para o servidor

### Opção A – usando ssh-copy-id (mais simples)

**No CLIENTE**
ssh-copy-id -i ~/.ssh/id_ed25519_servidor.pub usuario@IP_DO_SERVIDOR


### Opção B – se você já estiver logado no servidor

**No CLIENTE**
cat ~/.ssh/id_ed25519_servidor.pub


Copie toda a linha exibida.

**No SERVIDOR**
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys

Cole a chave em uma nova linha, salve e ajuste permissões:
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

## 4. Configurar o servidor para não aceitar login por senha nem root

**No SERVIDOR**

Não edite arquivos do cloud-init (50/60). Crie um arquivo próprio:
sudo nano /etc/ssh/sshd_config.d/99-custom.conf


Conteúdo:
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes


## 5. Verificar se o SSH aceita autenticação por chave

**No SERVIDOR**
sudo sshd -T | grep -E "pubkeyauthentication|passwordauthentication|permitrootlogin"


Saída esperada:

pubkeyauthentication yes
passwordauthentication no
permitrootlogin no


## 6. Reiniciar o serviço SSH

**No SERVIDOR**
sudo systemctl restart ssh
sudo systemctl status ssh


## 7. Testar login por chave

**No CLIENTE**
ssh -v usuario@IP_DO_SERVIDOR


Procure por:
- Offering public key: `~/.ssh/id_ed25519_servidor`
- Authentication succeeded

Se funcionou, o login por senha está definitivamente desativado.

## 8. Login Facilitado usando Chave SSH

### Facilitar login no Linux (Ghostty, terminal, etc.)

**No CLIENTE Linux**
nano ~/.ssh/config


Exemplo:

Host meu-servidor
HostName IP_DO_SERVIDOR
User usuario
IdentityFile ~/.ssh/id_ed25519_servidor
IdentitiesOnly yes

Permissões:
chmod 600 ~/.ssh/config


Conexão:

### Facilitar login no Windows

**Windows 10 / 11 (OpenSSH nativo)**

Copie a chave privada para:
C:\Users\SEU_USUARIO.ssh\


Crie ou edite:
C:\Users\SEU_USUARIO.ssh\config


Conteúdo:
Host meu-servidor
HostName IP_DO_SERVIDOR
User usuario
IdentityFile ~/.ssh/id_ed25519_servidor
Conectar:

**Alternativa: PuTTY**

Converter a chave com PuTTYgen e configurar em:
- Connection → SSH → Auth → Credentials

Salve a sessão.

## Boas práticas finais

chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519_servidor
chmod 644 ~/.ssh/id_ed25519_servidor.pub


## Checklist final

- Chave ED25519 criada
- Chave pública instalada no servidor
- Login por senha desativado
- Root bloqueado
- SSH reiniciado
- Login facilitado configurado

Dica: só remova chaves antigas (ssh-rsa) depois de confirmar que a nova funciona.

Se quiser, este guia pode ser convertido em PDF, README de projeto ou checklist corporativo.
