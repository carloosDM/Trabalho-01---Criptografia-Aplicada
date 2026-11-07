# Trabalho 01 — Criptografia Aplicada: Proteção, Assinaturas Digitais e Certificados

> **Disciplina:** Segurança e Auditoria de Sistemas de Informação  
> **Professor:** Joselito Júnior  
> **Empresa fictícia:** XYZW Tecnologia

Este repositório reúne os experimentos realizados no Trabalho 01. O objetivo é demonstrar, em um ambiente de laboratório, como proteger documentos digitais com **AES-256-GCM**, como gerar e verificar **assinaturas digitais RSA-PSS** e como validar certificados digitais X.509 emitidos por uma Autoridade Certificadora (CA).

Os exemplos foram preparados para serem reproduzidos por outros estudantes. Os testes utilizam somente documentos fictícios e arquivos locais.

## Visão geral dos experimentos

### Parte I — Criptografia e integridade com AES-GCM

Foi utilizado o AES-GCM por meio da biblioteca Python `cryptography`. O algoritmo protege a confidencialidade do documento e também autentica os dados: se o conteúdo cifrado for alterado ou se for utilizada uma chave incorreta, a descriptografia falha.

O fluxo realizado foi:

1. Criar o arquivo `documento_Joselito.txt`.
2. Gerar uma chave AES de 256 bits e um nonce de 12 bytes.
3. Cifrar o documento e salvar o resultado em `documento_Joselito.enc`.
4. Descriptografar o arquivo com a chave correta e recuperar `documento_recuperado.txt`.
5. Comparar o documento original com o recuperado.
6. Alterar um byte do conteúdo cifrado e repetir a descriptografia.
7. Tentar descriptografar o conteúdo original com uma chave incorreta.

**Resultados observados:**

```text
IGUAIS: documento recuperado corretamente
OK: adulteracao detectada!
OK: chave incorreta rejeitada!
```

### Parte II — Assinaturas digitais e certificados

Foi utilizado o OpenSSL para criar duas Autoridades Certificadoras independentes, emitir certificados X.509 e realizar assinaturas digitais com RSA-PSS e SHA-256.

Foram avaliados três cenários:

| Cenário | Assinatura digital | Cadeia do certificado | Decisão |
|---|---|---|---|
| A — Documento original | Válida | CA confiável | Aceitar |
| B — Documento alterado após a assinatura | Inválida | CA confiável | Rejeitar |
| C — Documento original | Válida | CA não confiável | Rejeitar |

**Interpretação:** a verificação matemática da assinatura e a validação da cadeia de certificação são verificações diferentes. Um documento só deve ser aceito quando atender aos critérios de integridade e confiança definidos pela aplicação.

---

## Requisitos

- Python 3 instalado.
- Biblioteca Python `cryptography`.
- OpenSSL instalado e disponível no terminal.
- Git Bash no Windows é recomendado para executar os comandos Bash apresentados.

Verifique as instalações:

```bash
python --version
openssl version
```

Instale a biblioteca Python:

```bash
python -m pip install cryptography
```

## Como reproduzir — Parte I (Python / AES-256-GCM)

Entre na pasta da Parte I:

```bash
cd parte1
```

Crie um documento de teste. Para reproduzir o exemplo, use:

```bash
printf 'DOCUMENTO DE TESTE — XYZW Tecnologia\n' > documento_Joselito.txt
```

### 1. Cifrar o documento

Execute:

```bash
python criptografia_gcm.py
```

O programa gera:

- `chave_aes_gcm.bin` — chave secreta AES-256;
- `nonce_aes_gcm.bin` — nonce utilizado pelo AES-GCM;
- `documento_Joselito.enc` — conteúdo cifrado e tag de autenticação.

### 2. Descriptografar

Execute:

```bash
python descriptografia_gcm.py
```

O resultado será salvo em `documento_recuperado.txt`. Compare os arquivos:

```bash
cmp -s documento_Joselito.txt documento_recuperado.txt && echo "IGUAIS: documento recuperado corretamente"
```

Se os arquivos forem iguais, a comparação apresentará a mensagem de sucesso.

### 3. Testar adulteração e chave incorreta

Execute:

```bash
python teste_falhas_gcm.py
```

O resultado esperado é:

```text
OK: adulteracao detectada!
OK: chave incorreta rejeitada!
```

O script altera um byte do arquivo cifrado e, em seguida, tenta descriptografá-lo. Também gera uma chave aleatória diferente da original e tenta utilizá-la. Em ambos os casos, o AES-GCM deve rejeitar a operação por falha de autenticação (`InvalidTag`).

---

## Como reproduzir — Parte II (OpenSSL / RSA / X.509)

Entre na pasta da Parte II:

```bash
cd ../parte2
```

O script abaixo automatiza a criação das duas CAs, dos certificados, a assinatura do documento e os testes dos três cenários:

```bash
bash parte2_certificados.sh
```

O script utiliza um arquivo de extensões real (`extensoes.cnf`) em vez de substituição de processo (`<(printf ...)`), pois essa última forma apresentou incompatibilidade no ambiente Git Bash utilizado no experimento.

### O que o script executa

1. Gera uma chave privada RSA de 2048 bits para a CA confiável.
2. Cria o certificado raiz autossinado da CA confiável (`CA:TRUE`).
3. Gera a chave privada do assinante e sua solicitação de certificado (CSR).
4. Emite o certificado do assinante pela CA confiável (`CA:FALSE`).
5. Cria `documento_xyzw.txt` e gera sua assinatura com SHA-256 e RSA-PSS.
6. Extrai a chave pública do certificado e verifica a assinatura e a cadeia — cenário A.
7. Altera o valor do documento e testa a assinatura original — cenário B.
8. Cria uma segunda CA independente e emite outro certificado para a mesma chave pública do assinante.
9. Demonstra que a cadeia emitida pela segunda CA não é validada pela CA confiável original.
10. Verifica a assinatura com a chave pública do segundo certificado — cenário C.

### Resultados esperados

**Cenário A:**

```text
Verified OK
assinante.crt: OK
```

**Cenário B:**

```text
Verification failure
```

**Cenário C:** a validação da cadeia deve falhar com uma mensagem semelhante a:

```text
error 20 at 0 depth lookup: unable to get local issuer certificate
```

Apesar disso, a verificação matemática da assinatura com a chave pública do certificado emitido pela CA não confiável deve apresentar:

```text
Verified OK
```

A falha de confiança ocorre porque a segunda CA não foi fornecida como âncora de confiança no comando `openssl verify -CAfile ca_confiavel.crt`.

---

## Arquivos do repositório

```text
.
├── README.md
├── parte1/
│   ├── criptografia_gcm.py
│   ├── descriptografia_gcm.py
│   └── teste_falhas_gcm.py
└── parte2/
    └── parte2_certificados.sh
```

Os arquivos gerados pelos experimentos (documentos, chaves, certificados e assinaturas) são criados localmente quando os scripts são executados.

## Observações de segurança

- Os exemplos são didáticos e destinados exclusivamente a um laboratório.
- **Não publique chaves privadas** (`*.key.pem`) nem a chave AES (`chave_aes_gcm.bin`) em repositórios públicos. O arquivo `.gitignore` deste projeto exclui esses arquivos.
- Em um sistema real, as chaves privadas devem ser armazenadas e protegidas adequadamente. Neste laboratório, a chave AES é gravada em arquivo apenas para permitir a reprodução do exercício.
- O certificado raiz criado no exercício só é considerado confiável porque foi explicitamente indicado como âncora de confiança por `-CAfile`. Ele não é uma CA pública reconhecida pelo sistema operacional ou pelos navegadores.
- A configuração do certificado do assinante reproduz o experimento com `extendedKeyUsage=emailProtection`. Essa extensão indica finalidade de proteção de e-mail. Para assinatura genérica de contratos, é recomendável omitir essa extensão e manter `keyUsage=digitalSignature`, emitindo novamente o certificado.
- O teste de cadeia apresentado é uma demonstração básica. Uma validação de produção também deve considerar validade temporal, finalidade do certificado, políticas aplicáveis, revogação e demais requisitos de confiança.
- O nonce do AES-GCM não deve ser reutilizado com a mesma chave. Cada operação de cifragem deve usar um nonce único.

## Referências e documentação

- PYCA / CRYPTOGRAPHY. *Authenticated encryption: AESGCM*. Disponível em: <https://cryptography.io/en/latest/hazmat/primitives/aead/>.
- OPENSSL. *openssl-genpkey — generate a private key*. Disponível em: <https://docs.openssl.org/3.5/man1/openssl-genpkey/>.
- OPENSSL. *openssl-req — certificate request and certificate generating utility*. Disponível em: <https://docs.openssl.org/3.5/man1/openssl-req/>.
- OPENSSL. *openssl-x509 — certificate display and signing utility*. Disponível em: <https://docs.openssl.org/3.5/man1/openssl-x509/>.
- OPENSSL. *openssl-dgst — message digest and signature*. Disponível em: <https://docs.openssl.org/3.5/man1/openssl-dgst/>.
- OPENSSL. *openssl-verify — certificate verification*. Disponível em: <https://docs.openssl.org/3.5/man1/openssl-verify/>.

---

**Trabalho acadêmico — Segurança e Auditoria de Sistemas de Informação.**  
**Empresa fictícia utilizada nos testes:** XYZW Tecnologia.
