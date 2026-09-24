# 💻 Tutorial Prático de Terminal: Detecção de Malware e Vulnerabilidades (Linux & Windows)

Aprenda a auditar conexões, processos e arquivos no seu sistema operacional usando a linha de comando para identificar ameaças, interpretar resultados normais vs. suspeitos e aplicar correções imediatas.

---

## 🚀 Introdução

* **O que este tutorial faz:** Ensina comandos essenciais para identificar infecções por *malware*, conexões não autorizadas, portas vulneráveis e falhas de configuração no Linux e no Windows.
* **Para quem é:** Qualquer usuário ou iniciante que busca monitorar e proteger seu sistema de forma prática.
* **Como abrir o terminal:**
  * **Linux:** Pressione `Ctrl + Alt + T` (ou abra o aplicativo Terminal).
  * **Windows:** Pressione `Win + X` e selecione **PowerShell** (ou **Terminal do Windows**).

---

## 🔍 Comandos de Detecção, Interpretação e Correção

---

### 1. Conexões Suspeitas
* **O que detecta:** Conexões de rede ativas com servidores externos desconhecidos ou portas de comunicação maliciosas.
* **Sintaxe & Exemplo Prático:**
  * 🐧 **Linux:**
    ```bash
    ss -tunap
    lsof -i
    ```
  * 🪟 **Windows (PowerShell):**
    ```powershell
    Get-NetTCPConnection -State Established
    netstat -ano | findstr ESTABLISHED
    ```
* **✅ Saída Esperada (Normal):** Conexões associadas apenas a programas conhecidos (navegadores, aplicativos de mensagens, atualizações do SO) conectadas a IPs de serviços legítimos ou ao endereço local (`127.0.0.1`).
* **🚨 Saída Suspeita (Com Problema):** Conexões ativas com IPs desconhecidos/estrangeiros em portas não usuais (ex: 4444, 6667, 1337) ou associadas a processos estranhos sem nome reconhecido.
* **🛠️ Como Corrigir a Vulnerabilidade:**
  1. Anote o ID do Processo (PID) associado à conexão suspeita.
  2. Encerre o processo imediatamente:
     * **Linux:** `kill -9 <PID>`
     * **Windows:** `Stop-Process -Id <PID> -Force`
  3. Bloqueie o endereço IP invasor no Firewall do sistema:
     * **Linux:** `sudo ufw deny out to <IP_SUSPEITO>`
     * **Windows:** `New-NetFirewallRule -DisplayName "Bloqueio_IP" -Direction Outbound -RemoteAddress <IP_SUSPEITO> -Action Block`
  4. Desconecte temporariamente o equipamento da internet até concluir a investigação.

---

### 2. Processos Estranhos e Consumo Excessivo
* **O que detecta:** Programas ocultos, consumo excessivo de processador por mineradores de criptomoedas não autorizados ou *scripts* maliciosos em segundo plano.
* **Sintaxe & Exemplo Prático:**
  * 🐧 **Linux:**
    ```bash
    ps aux | grep -v root
    top
    ```
  * 🪟 **Windows (PowerShell):**
    ```powershell
    Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
    tasklist /v
    ```
* **✅ Saída Esperada (Normal):** Uso de CPU e memória proporcional aos programas que você abriu conscientemente, com caminhos de execução conhecidos e nomes de processos identificáveis.
* **🚨 Saída Suspeita (Com Problema):** Processos consumindo perto de 100% de CPU continuamente, nomes aleatórios (ex: `x86_64_miner`, `svchost.exe` executando fora da pasta `System32`), ou executáveis rodando a partir de pastas temporárias (`/tmp`, `AppData\Local\Temp`).
* **🛠️ Como Corrigir a Vulnerabilidade:**
  1. Finalize o processo suspeito com base no seu PID:
     * **Linux:** `kill -9 <PID>`
     * **Windows:** `Stop-Process -Id <PID> -Force`
  2. Localize e remova o arquivo executável de origem no disco:
     * **Linux:** `rm -f /caminho/do/executavel_suspeito`
     * **Windows:** `Remove-Item C:\caminho\do\executavel_suspeito.exe -Force`
  3. Remova itens de inicialização automática associados no sistema (Cron/Systemd no Linux ou Registro/Agendador de Tarefas no Windows).

---

### 3. Arquivos Modificados Recentemente
* **O que detecta:** Alterações não autorizadas em arquivos de sistema ou novos executáveis/scripts maliciosos criados nas últimas 24 horas.
* **Sintaxe & Exemplo Prático:**
  * 🐧 **Linux:**
    ```bash
    find / -mtime -1 -type f 2>/dev/null
    ```
  * 🪟 **Windows (PowerShell):**
    ```powershell
    Get-ChildItem -Path C:\ -Recurse -File | Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-1)} -ErrorAction SilentlyContinue
    ```
* **✅ Saída Esperada (Normal):** Apenas arquivos de trabalho criados recentemente por você ou arquivos de cache/log atualizados por softwares legítimos em uso.
* **🚨 Saída Suspeita (Com Problema):** Arquivos executáveis ou *scripts* (`.sh`, `.exe`, `.ps1`, `.bat`) criados sem o seu conhecimento em diretórios temporários do sistema (`/tmp`, `/var/tmp`, `AppData\Local\Temp`).
* **🛠️ Como Corrigir a Vulnerabilidade:**
  1. Examine o conteúdo do arquivo com um leitor de texto sem executá-lo (ex: `head` no Linux ou `Get-Content` no Windows).
  2. Caso confirme a ameaça, exclua o arquivo imediatamente:
     * **Linux:** `rm -f /tmp/script_suspeito.sh`
     * **Windows:** `Remove-Item $env:TEMP\arquivo_suspeito.exe -Force`
  3. Altere as permissões do diretório temporário para proibir a execução de binários por usuários comuns.

---

### 4. Portas Abertas (Serviços Escutando)
* **O que detecta:** Serviços rodando sem o seu conhecimento e portas de rede expostas no computador local a conexões externas.
* **Sintaxe & Exemplo Prático:**
  * 🐧 **Linux:**
    ```bash
    ss -tulpn
    netstat -an | grep LISTEN
    ```
  * 🪟 **Windows (PowerShell):**
    ```powershell
    Get-NetTCPConnection -State Listen
    netstat -ano | findstr LISTENING
    ```
* **✅ Saída Esperada (Normal):** Nenhuma porta escutando conexões externas, ou apenas portas estritamente necessárias (como porta 80/443 para servidores web locais ou associadas ao endereço de *loopback* `127.0.0.1`).
* **🚨 Saída Suspeita (Com Problema):** Portas de administração ou de trojans conhecidos (ex: 22, 23, 135, 445, 3389, 4444) abertas na interface pública (`0.0.0.0` ou `::`) sem que você tenha configurado esses serviços.
* **🛠️ Como Corrigir a Vulnerabilidade:**
  1. Identifique o serviço ou programa atrelado à porta aberta.
  2. Encerrar ou desativar o serviço não necessário:
     * **Linux:** `sudo systemctl stop <nome_servico> && sudo systemctl disable <nome_servico>`
     * **Windows:** `Stop-Service -Name <nome_servico> && Set-Service -Name <nome_servico> -StartupType Disabled`
  3. Ative as regras de bloqueio do Firewall local para impedir tráfego nessas portas.

---

### 5. Integridade de Arquivos (Hashes)
* **O que detecta:** Alterações ou adulterações no código de programas baixados por meio da validação da assinatura digital hash.
* **Sintaxe & Exemplo Prático:**
  * 🐧 **Linux:**
    ```bash
    sha256sum /caminho/do/arquivo
    ```
  * 🪟 **Windows (PowerShell):**
    ```powershell
    Get-FileHash -Algorithm SHA256 C:\caminho\do\arquivo.exe
    ```
* **✅ Saída Esperada (Normal):** O código hash de 64 caracteres gerado no terminal é **exatamente idêntico** ao valor publicado na página oficial de download do desenvolvedor.
* **🚨 Saída Suspeita (Com Problema):** O código hash gerado é diferente daquele fornecido pelo desenvolvedor oficial, indicando que o arquivo foi modificado, corrompido ou infectado com código malicioso (*Trojan*).
* **🛠️ Como Corrigir a Vulnerabilidade:**
  1. Não execute o arquivo sob nenhuma hipótese.
  2. Apague o arquivo adulterado imediatamente do computador.
  3. Baixe o instalador novamente exclusivamente a partir do site oficial e verifique a hash do novo arquivo antes da instalação.

---

### 6. Permissões Perigosas e Elevação de Privilégios
* **O que detecta:** Arquivos ou serviços com permissões excessivas que permitem a usuários comuns executar ações com privilégios de Administrador (`root` / `SYSTEM`).
* **Sintaxe & Exemplo Prático:**
  * 🐧 **Linux (Arquivos com SUID ativado):**
    ```bash
    find / -perm -4000 -type f 2>/dev/null
    ```
  * 🪟 **Windows (Permissões de Arquivos e Serviços):**
    ```powershell
    Get-Acl C:\Caminho\Do\Arquivo | Format-List
    Get-WmiObject win32_service | Select-Object Name, PathName, StartMode
    ```
* **✅ Saída Esperada (Normal):** No Linux, apenas binários padrões de sistema (como `passwd`, `sudo`, `ping`). No Windows, caminhos de serviços apontando para diretórios protegidos (`System32`, `Program Files`) com acesso restrito a Administradores.
* **🚨 Saída Suspeita (Com Problema):** Binários desconhecidos em diretórios de usuários com a permissão SUID ativada, ou serviços do Windows em pastas graváveis por qualquer usuário ou com caminhos sem aspas (*Unquoted Service Paths*).
* **🛠️ Como Corrigir a Vulnerabilidade:**
  1. No Linux, remova a permissão SUID do arquivo suspeito:
     * `sudo chmod u-s /caminho/do/arquivo`
  2. No Windows, corrija as permissões da pasta usando `icacls` ou remova o serviço não autorizado:
     * `sc.exe delete <NomeDoServico>`

---

### 7. Varredura Básica de Malware
* **O que detecta:** Presença de assinaturas conhecidas de vírus, *ransomware*, *trojans* e *spyware* no sistema.
* **Sintaxe & Exemplo Prático:**
  * 🐧 **Linux (ClamAV):**
    ```bash
    clamscan -r -i /home
    ```
  * 🪟 **Windows (Windows Defender):**
    ```powershell
    Start-MpScan -ScanType QuickScan
    ```
* **✅ Saída Esperada (Normal):** Relatório sem nenhuma ameaça encontrada (`Infected files: 0` no ClamAV ou ausência de alertas de ameaça no Windows Defender).
* **🚨 Saída Suspeita (Com Problema):** Notificação de infecção (ex: `FOUND Trojan.Agent`, `Ransomware.Win32`, `Win32/CoinMiner`).
* **🛠️ Como Corrigir a Vulnerabilidade:**
  1. Envie a ameaça para a quarentena ou remova o arquivo detectado:
     * **Linux:** `clamscan --remove=yes -r /home`
     * **Windows:** `Remove-MpThreat`
  2. Execute uma varredura completa (*FullScan*) em Modo de Segurança.
  3. Altere imediatamente as senhas de todas as contas salvas no equipamento infectado.

---

### 8. Logs Suspeitos do Sistema
* **O que detecta:** Erros de sistema, falhas consecutivas de autenticação ou tentativas de invasão por força bruta (*Brute Force*).
* **Sintaxe & Exemplo Prático:**
  * 🐧 **Linux:**
    ```bash
    journalctl -p err -n 20
    ```
  * 🪟 **Windows (PowerShell):**
    ```powershell
    Get-EventLog -LogName System -EntryType Error -Newest 20
    Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} -MaxEvents 10
    ```
* **✅ Saída Esperada (Normal):** Erros pontuais de inicialização de hardware/drivers e ausência de falhas repetidas de login.
* **🚨 Saída Suspeita (Com Problema):** Dezenas ou centenas de registros consecutivos de falha de login (Event ID 4625 no Windows ou falhas SSH no Linux) em curtos intervalos de tempo.
* **🛠️ Como Corrigir a Vulnerabilidade:**
  1. Instale e configure uma ferramenta de bloqueio automático contra força bruta no Linux (ex: **Fail2ban**):
     * `sudo apt install fail2ban`
  2. No Windows, configure a política de bloqueio de conta após 5 tentativas incorretas:
     * `net accounts /lockoutthreshold:5`
  3. Desative a autenticação por senha para acessos remotos (SSH), exigindo apenas chaves públicas criptográficas.

---

## 🚨 Passo a Passo Rápido: "Suspeito que minha máquina está infectada. O que faço?"

1. **Passo 1 — Isolar e verificar conexões ativas:** Execute `ss -tunap` (Linux) ou `Get-NetTCPConnection` (Windows). Se encontrar IPs desconhecidos, encerre o processo com `kill` / `Stop-Process` e desconecte a internet.
2. **Passo 2 — Localizar e matar processos estranhos:** Rode `top` / `ps aux` (Linux) ou `Get-Process` (Windows). Identifique processos com alto uso de CPU e encerre-os pelo PID.
3. **Passo 3 — Remover arquivos temporários maliciosos:** Execute a busca por arquivos modificados recentemente nas pastas `/tmp` ou `AppData\Local\Temp` e remova-os.
4. **Passo 4 — Executar varredura antivírus completa:** Rode `clamscan -r -i /home` (Linux) ou `Start-MpScan -ScanType FullScan` (Windows) para eliminar remanescentes do malware.
5. **Passo 5 — Verificar e corrigir os logs de erro:** Examine `journalctl -p err` (Linux) ou `Get-EventLog` (Windows) e ative regras de firewall/Fail2ban para evitar novas tentativas de invasão.

---

## ⚠️ Sinais de Alerta (Checklist)

* ⚠️ Processador/RAM operando acima de 90% sem aplicativos pesados abertos.
* ⚠️ Endereços IP desconhecidos mantendo conexões ativas de saída.
* ⚠️ Arquivos criados recentemente em pastas temporárias (`/tmp`, `AppData\Local\Temp`).
* ⚠️ Portas desconhecidas em estado `LISTEN` / `Listening` expostas para a rede.
* ⚠️ Dezenas de tentativas de autenticação com falha registradas nos logs do sistema.

---

## 📊 Tabela Resumo dos Comandos

| O que detecta | Comando Linux | Comando Windows (PowerShell) | Ação de Correção Rápida |
| :--- | :--- | :--- | :--- |
| **Conexões Ativas** | `ss -tunap` | `Get-NetTCPConnection -State Established` | Enerrar PID (`kill`) e bloquear IP no Firewall |
| **Processos Estranhos** | `ps aux` / `top` | `Get-Process \| Sort-Object CPU -Descending` | Matar processo e excluir executável do disco |
| **Arquivos Recentes** | `find /tmp -mtime -1` | `Get-ChildItem $env:TEMP -Recurse ...` | Apagar arquivos suspeitos não criados por você |
| **Portas Abertas** | `ss -tulpn` | `Get-NetTCPConnection -State Listen` | Parar o serviço não utilizado e ativar o Firewall |
| **Integridade (Hash)** | `sha256sum <arquivo>` | `Get-FileHash -Algorithm SHA256 <arquivo>` | Deletar arquivo se a hash for diferente da oficial |
| **Varredura de Malware**| `clamscan -r -i /home` | `Start-MpScan -ScanType QuickScan` | Enviar ameaças para quarentena / remoção |
| **Logs de Erros** | `journalctl -p err` | `Get-EventLog -LogName System -EntryType Error` | Ativar Fail2ban ou política de bloqueio de conta |

---

*Nota de Segurança: Após remediar qualquer infecção ou vulnerabilidade, altere imediatamente todas as suas senhas importantes e ative a autenticação em dois fatores (2FA) em suas contas.*
