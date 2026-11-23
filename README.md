# 🔐 Simulando Testes de Força Bruta com Medusa e Kali Linux

Projeto desenvolvido para o desafio prático do **Bootcamp Santander Cibersegurança do Zero à Prática** na plataforma **DIO (Digital Innovation One)**.

> ⚠️ **Aviso Legal e Ético:** Todo o conteúdo deste repositório tem finalidade estritamente **educacional** e foi elaborado para estudo em ambiente de laboratório controlado e isolado (máquinas virtuais locais). Nunca execute testes de auditoria ou validação de senhas em sistemas sem autorização prévia e explícita.

---

## 📋 Sobre o Desafio

O objetivo deste projeto é compreender na prática como funcionam testes automatizados de adivinhação de senhas — como **Brute Force**, **Password Spraying** e **Credential Stuffing** — utilizando a ferramenta **Medusa** no **Kali Linux** contra um ambiente intencionalmente vulnerável (**Metasploitable 2** e **DVWA**), além de propor medidas eficazes de **prevenção e mitigação**.

### 🎯 Objetivos de Aprendizagem
- Compreender as diferenças entre *Brute Force*, *Password Spraying* e *Credential Stuffing*.
- Configurar um laboratório seguro e isolado com VirtualBox (`Host-Only`).
- Utilizar ferramentas de auditoria como **Nmap**, **Enum4linux**, **Medusa** e **smbclient** em ambiente controlado.
- Estruturar listas de usuários e senhas (*wordlists*) simples para testes de segurança.
- Propor medidas de mitigação para proteger serviços como FTP, aplicações Web e SMB.

---

## 📚 Conceitos Fundamentais

| Técnica | Como funciona | Principal risco / característica |
| :--- | :--- | :--- |
| **Brute Force (Força Bruta)** | Testa diversas combinações de senhas para um ou mais usuários utilizando *wordlists*. | Gera alto volume de falhas por conta e costuma disparar bloqueios (*account lockout*). |
| **Password Spraying** | Testa **uma única senha comum** contra **vários usuários** ao mesmo tempo. | Evita o bloqueio de contas individuais por excesso de tentativas, passando mais despercebido. |
| **Credential Stuffing** | Reutiliza pares de e-mail/usuário e senha obtidos em vazamentos anteriores contra outros serviços. | Explora o hábito de reutilização de senhas entre plataformas diferentes. |

---

## 🗂️ Estrutura do Repositório

```text
brute-force-medusa-kali/
├── README.md                 # Documentação completa do laboratório e mitigações
├── .gitignore                # Arquivos ignorados pelo Git
├── images/                   # Pasta preparada para capturas de tela das VMs (opcional)
│   └── .gitkeep
└── wordlists/                # Listas de exemplo usadas nos testes controlados
    ├── users.txt             # Lista de nomes de usuários comuns (Metasploitable 2)
    ├── passwords.txt         # Lista de senhas fracas/comuns
    └── smb_users.txt         # Lista mista simulando cenário corporativo no SMB
```

> ℹ️ **Nota sobre o laboratório:** Os endereços IP (como `192.168.56.102`) e comandos abaixo são exemplos ilustrativos baseados em uma rede interna `Host-Only` do VirtualBox. A pasta `images/` está reservada para inclusão de prints caso você execute as máquinas virtuais localmente.

---

## 🛠️ Configuração do Ambiente de Laboratório

Para garantir segurança total durante os estudos, o laboratório utiliza duas Máquinas Virtuais (VMs) isoladas da internet pública através de uma rede interna (`Host-Only Adapter` no VirtualBox):

| Máquina Virtual | Função no Laboratório | Sistema / Aplicação |
| :--- | :--- | :--- |
| **Kali Linux** | Máquina de Auditoria | Ferramentas `nmap`, `enum4linux`, `medusa`, `smbclient` |
| **Metasploitable 2** | Máquina Alvo Vulnerável | Serviços FTP, SMB e aplicação Web **DVWA** |

### 1. Verificando a conectividade e portas abertas
No Kali Linux, após identificar os IPs na rede interna (por exemplo, `192.168.56.101` para o Kali e `192.168.56.102` para o Metasploitable 2), verificamos os serviços ativos no alvo com o `nmap`:

```bash
nmap -sV 192.168.56.102
```

Principais serviços observados no ambiente de teste:
- **Porta 21/tcp (FTP):** Serviço de transferência de arquivos.
- **Porta 80/tcp (HTTP):** Servidor web hospedando o **DVWA** (*Damn Vulnerable Web Application*).
- **Portas 139 e 445/tcp (SMB):** Serviço de compartilhamento de arquivos em rede.

---

## 🧪 Cenários de Teste Documentados

> 💡 **Dica rápida sobre as opções do Medusa:** letras **maiúsculas** (`-U`, `-P`) recebem **arquivos de lista** (*wordlists*), enquanto letras **minúsculas** (`-u`, `-p`) recebem um **único valor** direto no comando.

### Cenário 1: Auditoria de Senhas no Serviço FTP
O protocolo FTP (sem criptografia) é um alvo clássico para validação de credenciais fracas. Utilizando o módulo `ftp` do Medusa com as listas em `wordlists/`:

```bash
medusa -h 192.168.56.102 -U wordlists/users.txt -P wordlists/passwords.txt -M ftp
```

**Parâmetros utilizados:**
- `-h`: IP do host alvo no laboratório.
- `-U`: Caminho para o arquivo com a lista de usuários.
- `-P`: Caminho para o arquivo com a lista de senhas.
- `-M`: Módulo do serviço a ser testado (`ftp`).

---

### Cenário 2: Automação de Tentativas em Formulário Web (DVWA)
Em aplicações web como o DVWA (configurado no nível *Low* para fins didáticos), o Medusa pode testar formulários de login HTTP usando o módulo `web-form`:

```bash
medusa -h 192.168.56.102 -u admin -P wordlists/passwords.txt -M web-form -m FORM:"/dvwa/login.php" -m DENY-SIGNAL:"Login failed" -m FORM-DATA:"post?username=&password=&Login=Login"
```

**Parâmetros utilizados:**
- `-u`: Usuário único a ser testado (`admin`).
- `-P`: Caminho para a lista de senhas (`wordlists/passwords.txt`).
- `-M web-form`: Módulo do Medusa para formulários HTTP.
- `-m FORM`: Caminho relativo do formulário de login no servidor web.
- `-m DENY-SIGNAL`: Texto exato exibido na página quando o login falha (se esse texto não aparecer na resposta, o Medusa considera que houve sucesso).
- `-m FORM-DATA`: Método HTTP (`post` ou `get`) seguido dos campos do formulário na ordem usuário e senha.

**Observações e limitações técnicas:**
- Caso o teste seja direcionado à página interna do módulo *Brute Force* do DVWA (`/dvwa/vulnerabilities/brute/`), o formulário utiliza o método `GET`, depende de cookie de sessão autenticado (`PHPSESSID`) e exibe a mensagem `"Username and/or password incorrect."`.
- Aplicações web modernas costumam usar tokens **CSRF**, cookies de sessão obrigatórios, **CAPTCHA** e limites de requisição (*Rate Limiting*). Isso evidencia as limitações do módulo `web-form` do Medusa frente a interfaces web modernas e explica a ocorrência de falsos positivos quando `DENY-SIGNAL` não corresponde exatamente à resposta do servidor.

---

### Cenário 3: Enumeração SMB, *Password Spraying* e Validação

Este cenário simula um ambiente corporativo mal configurado, no qual o arquivo `wordlists/smb_users.txt` combina contas padrão do sistema (`msfadmin`, `service`) com contas organizacionais fictícias (`suporte`, `financeiro`, `rh`, `ti_admin`, `backup`).

1. **Enumeração inicial de usuários e compartilhamentos SMB:**
Antes de testar senhas, identificamos usuários e recursos expostos no alvo:

```bash
enum4linux -a 192.168.56.102
```
*(Alternativamente, via Nmap: `nmap --script smb-enum-users -p 445 192.168.56.102`)*

2. **Executando *Password Spraying* com o Medusa:**
Em vez de testar muitas senhas em uma conta só, testamos uma senha comum (`msfadmin`) contra toda a lista de usuários:

```bash
medusa -h 192.168.56.102 -U wordlists/smb_users.txt -p "msfadmin" -M smbnt
```

**Parâmetros utilizados:**
- `-U`: Arquivo com a lista de usuários enumerados (`wordlists/smb_users.txt`).
- `-p`: Senha única testada contra todos os usuários da lista.
- `-M smbnt`: Módulo do Medusa para autenticação em serviços SMB/NT.

3. **Validando o acesso descoberto com `smbclient`:**
Após identificar uma credencial válida, listamos os compartilhamentos disponíveis e testamos o acesso:

```bash
smbclient -L //192.168.56.102 -U msfadmin
```
- `-L`: Lista os diretórios compartilhados pelo servidor.
- `-U`: Define o usuário utilizado na autenticação.

Para conectar diretamente a um compartilhamento específico (como `/tmp`):

```bash
smbclient //192.168.56.102/tmp -U msfadmin
```

---

## 🛡️ Medidas de Prevenção e Mitigação

A parte mais importante de uma auditoria de segurança é saber **como corrigir** as vulnerabilidades encontradas. Abaixo estão as principais defesas contra ataques de força bruta, *password spraying* e *credential stuffing*:

1. **Bloqueio de Conta (*Account Lockout*) e *Rate Limiting*:**
   - Limitar a quantidade de tentativas de login falhas consecutivas (ex.: bloquear temporariamente após 5 tentativas).
   - Aplicar atrasos progressivos (*exponential backoff*) ou ferramentas como **Fail2Ban**.
2. **Autenticação Multifator (MFA / 2FA):**
   - Mesmo que uma senha seja descoberta ou vazada, o atacante não consegue acessar a conta sem o segundo fator de verificação.
3. **Política de Senhas Fortes e Únicas:**
   - Exigir senhas longas e não reutilizadas entre serviços, eliminando senhas padrão de fábrica (`msfadmin`, `admin`, `123456`).
4. **Uso de Protocolos Seguros e Hardening:**
   - Substituir FTP por **SFTP** ou **FTPS** (com tráfego criptografado).
   - Restringir o acesso ao protocolo **SMB** apenas à rede interna necessária, desativar versões antigas (como SMBv1) e desabilitar sessões anônimas/nulas.
5. **Monitoramento e Logs:**
   - Configurar alertas no SIEM ou nos logs do servidor para identificar picos de falhas de autenticação — especialmente múltiplas falhas em contas diferentes vindas de um mesmo IP, comportamento típico de *Password Spraying*.

---

## 📚 Referências e Materiais de Apoio
- [Kali Linux – Site Oficial](https://www.kali.org/)
- [Medusa – Documentação Oficial](http://www.foofus.net/jmk/medusa/medusa.html)
- [Nmap – Manual Oficial](https://nmap.org/book/)
- [DVWA – Damn Vulnerable Web Application](http://www.dvwa.co.uk/)
- [Guia de Markdown do GitHub](https://docs.github.com/pt/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
