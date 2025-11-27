# CTF Write-up: BrineBreaker (Pickle Scan Evasion)

Este repositório contém a Prova de Conceito (PoC) e o *write-up* da solução para o desafio **"BrineBreaker: Pickle Scan Evasion"** da plataforma **CyberWarFare**.

O objetivo do desafio era explorar uma vulnerabilidade de **Desserialização Insegura** em uma aplicação Python, contornando um *scanner* de segurança estático que bloqueava payloads maliciosos óbvios.

---

## ⚠️ AVISO LEGAL

**Este código é estritamente educacional e foi desenvolvido para resolver um desafio de CTF (Capture The Flag) autorizado.**
Não utilize técnicas de ofuscação ou exploração em sistemas sem permissão explícita. A desserialização insegura é uma vulnerabilidade crítica; a correção recomendada é nunca desserializar dados não confiáveis.

---

## 🔬 A Vulnerabilidade: Python Pickle Deserialization

O módulo `pickle` do Python permite serializar e desserializar estruturas de objetos complexos. No entanto, ele é inerentemente inseguro.

A máquina virtual do `pickle` permite a execução de código arbitrário durante o processo de desserialização através do método mágico `__reduce__`. Se uma aplicação aceita dados *pickled* de um usuário e os processa com `pickle.load()`, um atacante pode forçar a aplicação a executar comandos do sistema.

### O Desafio (O Scanner)

Neste cenário específico ("BrineBreaker"), a aplicação implementava um **Scanner Estático** antes de desserializar o objeto. O scanner buscava por assinaturas de texto simples comuns em exploits, como:
* `os.system`
* `subprocess.run`
* `/bin/sh`

Qualquer payload contendo essas strings era rejeitado.

---

## 🛠️ A Solução: Bypass via Ofuscação Base64

Para contornar a análise estática, desenvolvi um *payload* que não contém nenhuma das strings proibidas em texto claro.

### A Estratégia

Utilizei uma técnica de "inception" (execução dentro de execução) com codificação **Base64**:

1.  **Encapsulamento:** O comando malicioso (RCE) é escrito normalmente.
2.  **Codificação:** O comando é convertido para Base64. Isso transforma strings legíveis (ex: `os.system`) em strings alfanuméricas inócuas para o scanner.
3.  **Montagem do Payload (`__reduce__`):** O objeto malicioso instrui o interpretador Python a usar a função nativa `exec()`.
4.  **Decodificação em Tempo de Execução:** Dentro do `exec()`, passamos uma instrução para importar `base64` e decodificar nosso payload *apenas no momento da execução*.

---

## 🚀 Como Usar

#### 1. Clone o repositório
```bash
git clone [https://github.com/Luan-Garcia/pickle-scan-bypass-poc.git](https://github.com/Luan-Garcia/pickle-scan-bypass-poc.git)
cd pickle-scan-bypass-poc
```
#### 2. Configure o Alvo (Opcional)
```python
TARGET_COMMAND = 'cat /flag.txt' # Ou seu comando de Reverse Shell
```

#### 3. Gere o Arquivo Malicioso
```python
python3 exploit.py
```
---

## 🛡️Mitigação 
A única maneira segura de lidar com dados não confiáveis é não usar Pickle.

Use JSON: Para transferência de dados, prefira formatos como JSON (json.loads), que não permitem execução de código arbitrário por design.

Assinatura HMAC: Se o uso de Pickle for estritamente necessário (ex: manter estado interno de sessão legado), os dados devem ser assinados criptograficamente com uma chave secreta (HMAC) para garantir que não foram adulterados ou gerados pelo cliente.

Autor: Luan Garcia - Security Researcher | Pentester | Exploit Developer
