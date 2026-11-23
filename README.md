# 🔐 Simulando Testes de Força Bruta com Medusa e Kali Linux

Projeto desenvolvido para o desafio prático do **Bootcamp Santander Cibersegurança do Zero à Prática** na plataforma **DIO (Digital Innovation One)**.

> ⚠️ **Aviso Legal e Ético:** Todo o conteúdo deste repositório tem finalidade estritamente **educacional** e foi elaborado para estudo em ambiente de laboratório controlado e isolado (máquinas virtuais locais). Nunca execute testes de auditoria ou validação de senhas em sistemas sem autorização prévia e explícita.

---

## 📋 Sobre o Desafio

O objetivo deste projeto é compreender na prática como funcionam ataques automatizados de adivinhação de senhas — como **Brute Force** e **Password Spraying** — utilizando a ferramenta **Medusa** no **Kali Linux** contra um ambiente intencionalmente vulnerável (**Metasploitable 2** e **DVWA**), e aprender como implementar medidas eficazes de **prevenção e defesa**.

### 🎯 Objetivos de Aprendizagem
- Compreender os conceitos de *Brute Force*, *Password Spraying* e *Credential Stuffing*.
- Configurar um laboratório seguro e isolado com VirtualBox (`Host-Only`).
- Utilizar ferramentas de auditoria como **Nmap**, **Medusa** e **smbclient** em ambiente controlado.
- Estruturar listas de usuários e senhas (*wordlists*) simples para testes de segurança.
- Propor medidas de mitigação para proteger serviços como FTP, aplicações Web e SMB.

---

## 🗂️ Estrutura do Repositório

```text
dio-brute-force-medusa-kali/
├── README.md                 # Documentação completa do laboratório e mitigações
├── .gitignore                # Arquivos ignorados pelo Git
├── images/                   # Pasta para prints e evidências dos testes (opcional)
│   └── .gitkeep
└── wordlists/                # Listas de exemplo usadas nos testes controlados
    ├── users.txt             # Lista de nomes de usuários comuns
    ├── passwords.txt         # Lista de senhas fracas/comuns
    └── smb_users.txt         # Lista de usuários para cenário SMB
```

---

## 🛠️ Configuração do Ambiente de Laboratório

Para garantir segurança total durante os estudos, o laboratório utiliza duas Máquinas Virtuais (VMs) isoladas da internet pública através de uma rede interna (`Host-Only Adapter` no VirtualBox):

| Máquina Virtual | Função no Laboratório | Sistema / Aplicação |
| :--- | :--- | :--- |
| **Kali Linux** | Máquina de Auditoria | Ferramentas `nmap`, `medusa`, `smbclient` |
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

**Observações importantes:**
- Aplicações web modernas costumam usar tokens **CSRF**, proteção contra automação (CAPTCHA), limites de requisição (*Rate Limiting*) e cabeçalhos dinâmicos, o que limita ferramentas clássicas como o Medusa e gera falsos positivos caso o sinal de erro (`DENY-SIGNAL`) não seja configurado corretamente.

---

### Cenário 3: Enumeração SMB e *Password Spraying*
Diferente do *Brute Force* tradicional (que testa muitas senhas para um usuário), o **Password Spraying** testa **uma única senha comum** contra **vários usuários** ao mesmo tempo, reduzindo a chance de bloquear contas por excesso de tentativas.

1. **Testando uma senha padrão na lista de usuários SMB (`wordlists/smb_users.txt`):**

```bash
medusa -h 192.168.56.102 -U wordlists/smb_users.txt -p "msfadmin" -M smbnt
```

2. **Validando o acesso descoberto com `smbclient`:**

```bash
smbclient -L //192.168.56.102 -U msfadmin
```

---

## 🛡️ Medidas de Prevenção e Mitigação

A parte mais importante de uma auditoria de segurança é saber **como corrigir** as vulnerabilidades encontradas. Abaixo estão as principais defesas contra ataques de força bruta e *password spraying*:

1. **Bloqueio de Conta (*Account Lockout*) e *Rate Limiting*:**
   - Limitar a quantidade de tentativas de login falhas consecutivas (ex: bloquear temporariamente após 5 tentativas).
   - Aplicar atrasos progressivos (*exponential backoff*) ou ferramentas como **Fail2Ban**.
2. **Autenticação Multifator (MFA / 2FA):**
   - Mesmo que uma senha seja descoberta ou vazada, o atacante não consegue acessar a conta sem o segundo fator de verificação.
3. **Política de Senhas Fortes:**
   - Exigir senhas longas, únicas e complexas, evitando senhas padrão de fábrica (`msfadmin`, `admin`, `123456`).
4. **Uso de Protocolos Seguros:**
   - Substituir FTP por **SFTP** ou **FTPS** (com tráfego criptografado).
   - Restringir o acesso ao protocolo **SMB** apenas à rede interna necessária e desativar versões antigas (como SMBv1).
5. **Monitoramento e Logs:**
   - Configurar alertas no SIEM ou nos logs do servidor para identificar picos de falhas de autenticação (especialmente várias falhas em contas diferentes vindas do mesmo IP, típico de *Password Spraying*).

---

## 📚 Referências e Materiais de Apoio
- [Kali Linux – Site Oficial](https://www.kali.org/)
- [Medusa – Documentação Oficial](http://www.foofus.net/jmk/medusa/medusa.html)
- [Nmap – Manual Oficial](https://nmap.org/book/)
- [DVWA – Damn Vulnerable Web Application](http://www.dvwa.co.uk/)
- [Guia de Markdown do GitHub](https://docs.github.com/pt/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
