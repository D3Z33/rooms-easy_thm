<div align="center">

# 🎩 TryHackMe – W1seGuy (Writeup)

*"A sabedoria está na simplicidade dos XORs."* 🔑✨  

<p align="center">
  <img src="https://readme-typing-svg.herokuapp.com?font=Fira+Code&size=22&pause=1000&color=00FF00&width=600&lines=Linux+Exploitation;Cryptography;XOR+Brute+Force;CTF+Walkthrough" alt="Typing SVG" />
</p>

</div>

---

<div align="center">

## 📌 Informações da Máquina

- **Categoria:** Linux | Crypto  
- **Dificuldade:** 🟢 Easy  
- **Tempo Estimado:** ⏱️ 5 min  
- **Status:** ✅ Rooted com sucesso  

</div>

---

<div align="center">
    
## 🗺️ Roadmap da Exploração

</div>

```mermaid
graph TD
A[Análise do Código Fonte] --> B[XOR Encoding]
B --> C[Recuperar a Chave com Known-Plaintext]
C --> D[Flag 1 Decifrada]
D --> E[Envio da Chave Correta]
E --> F[Flag 2 Liberada]
F --> G[Brute Force Opcional]
````

---

## 🔍 Analisando o Código Fonte

Logo de início recebemos um **arquivo Python** que define um servidor na porta `1337`.

Trecho principal:

```python
class RequestHandler(socketserver.BaseRequestHandler):
    def handle(self):
        start(self.request)

if __name__ == '__main__':
    socketserver.ThreadingTCPServer.allow_reuse_address = True
    server = socketserver.ThreadingTCPServer(('0.0.0.0', 1337), RequestHandler)
    server.serve_forever()
```

Ou seja, todas as conexões são tratadas pela função `start()`.

---

### Função `start()`

1. Gera uma **chave aleatória de 5 caracteres**:

```python
res = ''.join(random.choices(string.ascii_letters + string.digits, k=5))
key = str(res)
```

2. Chama a função `setup()` para XOR da flag:

```python
hex_encoded = setup(server, key)
```

3. Envia o resultado para o cliente e pede a chave:

```python
send_message(server, "This XOR encoded text has flag 1: " + hex_encoded + "\n")
send_message(server,"What is the encryption key? ")
```

Se a chave estiver correta → retorna a **flag 2**.
Se não → mensagem de erro.

---

### Função `setup()`

```python
def setup(server, key):
    flag = 'THM{thisisafakeflag}' 
    xored = ""
    for i in range(0,len(flag)):
        xored += chr(ord(flag[i]) ^ ord(key[i%len(key)]))
    hex_encoded = xored.encode().hex()
    return hex_encoded
```

O que acontece:

* Pega cada caractere da flag.
* Faz XOR com o caractere correspondente da chave (cíclica).
* Retorna o resultado em **hexadecimal**.

---

## 📐 Propriedade do XOR

Dado:

```
F ⊕ K = E
```

Temos também:

```
F ⊕ E = K
K ⊕ E = F
```

Ou seja, conhecendo **flag inicial** (`THM{`) e o **texto cifrado**, podemos recuperar os primeiros bytes da chave.

---

## ⚡ Execução na Prática

### Conectividade

```bash
root@ip-10-201-72-242:~# ALVO="10.201.109.208"
root@ip-10-201-72-242:~# ping -c 3 $ALVO | grep -o "ttl=[0-9]*"
ttl=64
ttl=64
ttl=64
```

### Conexão no serviço

```bash
root@ip-10-201-72-242:~# nc $ALVO 1337
This XOR encoded text has flag 1: 352e220c145007031910241e1b361015520c1c0720081d44050d2a161f311312164711131e200519
What is the encryption key?
```

---

## 🔑 Recuperando a Chave

Sabemos que a flag começa com `THM{`.
Podemos usar isso para extrair os 4 primeiros bytes da chave:

```python
>>> from pwn import xor
>>> flag_encriptada = bytes.fromhex('352e220c145007031910241e1b361015520c1c0720081d44050d2a161f311312164711131e200519')
>>> xor(flag_encriptada[:4], b"THM{")
b'Or0R'
```

Ou seja, os primeiros 4 caracteres da chave = **`Or0R`**.

---

### Último caractere da chave

A flag tem 40 caracteres.
O último é `}`, e o último byte cifrado é `0x3f`.

```python
>>> xor(flag_encriptada[-1:], b"}")
b'x'
```

Portanto, a chave completa = **`Or0Rx`**

---

## 🚩 Flags

### Flag 1

```python
>>> xor(flag_encriptada, b"Or0Rx")
b'THM{p1alntExtAtt4ckcAnr3alLyhUrty0urxOr}'
```

✅ **Flag 1:**

```
THM{*****}
```

---

### Flag 2

Basta enviar a chave correta no prompt do servidor:

```
What is the encryption key? Or0Rx
Congrats! That is the correct key! Here is flag 2: THM{*****}
```

✅ **Flag 2:**

```
THM{*****}
```

---

## 🧪 Brute Force (Opcional)

Também é possível resolver por brute-force, testando todos os valores possíveis para o 5º byte da chave:

```python
from pwn import xor
import string

cifrado = bytes.fromhex("1d037d3c32782a5c29360c334406363d7f532c2108254274232507492f173b3f4977373b337f353f")

chave_inicial = xor(b"THM{", cifrado[:4])  # b"IK0G"

for b5 in range(256):
    chave = chave_inicial + bytes([b5])
    chave_repetida = (chave * ((len(cifrado)+len(chave)-1)//len(chave)))[:len(cifrado)]
    decifrado = xor(cifrado, chave_repetida)

    try:
        print(chave, ":", decifrado.decode("ascii"))
    except UnicodeDecodeError:
        print(chave, ":", decifrado)
```

Execução retorna, entre outros:

```
b'IK0GB' : THM{*****}
```

---

## 🧩 Fluxo Final

1. Analisamos o código-fonte.
2. Identificamos a criptografia XOR com chave de 5 bytes.
3. Usamos **known-plaintext attack** para descobrir 4 bytes da chave.
4. Recuperamos o último byte com o último caractere `}`.
5. Deciframos a Flag 1.
6. Enviamos a chave correta → Flag 2.
7. Extra: brute-force confirma a chave correta.

---

## 🎉 Conclusão

Técnicas aplicadas:

* **Crypto analysis** (XOR properties)
* **Known-plaintext attack**
* **Brute-force validation**

---

<div align="center">

![Snake animation](https://github.com/Platane/snk/raw/output/github-contribution-grid-snake.svg)

</div>

