

# Trabalho 5 (T5) — Pipeline CI Mínimo e Reprodutível

**Disciplina:** Cloud Computing e DevOps Avançado  
**Curso:** CST em Sistemas para Internet | 6º Período  
**Professor:** Ranghetti  
**Projeto:** Aplicação Biblioteca  

---

## 👥 Integrantes do Grupo

* [Nome Completo do Integrante 1] - [RA/Matrícula]
* [Nome Completo do Integrante 2] - [RA/Matrícula]
* [Nome Completo do Integrante 3] - [RA/Matrícula]
* [Nome Completo do Integrante 4] - [RA/Matrícula]

---

## 🚀 Instruções de Execução

### Pré-requisitos (Execução Local)
* **Java JDK:** Versão 21 (distribuição recomendada: Eclipse Temurin)
* **Git:** Para clonagem e controlo de versão
* *(Nota: Não é necessário instalar o Maven previamente, pois o projeto utiliza o **Maven Wrapper** `./mvnw`)*

### Passo a Passo Local
1. Clone este repositório e aceda à pasta do projeto:
   ```bash
   git clone <URL_DO_REPOSITORIO>
   cd <NOME_DO_DIRETORIO>
   ```


2. Garanta permissão de execução ao Maven Wrapper (em ambientes Linux/macOS):

```bash
chmod +x mvnw
```


3. Defina a variável de ambiente de teste e execute o ciclo completo de limpeza, compilação, testes e empacotamento:

```bash
export API_SECURITY_TOKEN_SECRET="ci-test-secret-0123456789abcdef0123456789abcdef"
./mvnw --batch-mode --no-transfer-progress clean verify
```


*(No Windows PowerShell, utilize `$env:API_SECURITY_TOKEN_SECRET="ci-test-secret-0123456789abcdef0123456789abcdef"` antes de correr `./mvnw.cmd ...`)*

### Execução no Pipeline (GitHub Actions)

O pipeline está definido em `.github/workflows/ci.yml` (nomeado como **CI**) e é iniciado automaticamente conforme os gatilhos configurados ou manualmente através da interface do GitHub.

---

## ⚙️ Anatomia e Identificação do Pipeline

### 1. Gatilho (Trigger)

* **`push`:** Disparado automaticamente a cada envio de commits para as branches `master` ou `main`.
* **`pull_request`:** Disparado ao abrir ou atualizar um Pull Request direcionado às branches `master` ou `main`.
* **`workflow_dispatch`:** Permite o acionamento manual do pipeline diretamente pela aba *Actions* no GitHub.

### 2. Etapas do Pipeline (Job: `build-test-package`)

1. **Check out source (`actions/checkout@v6`):** Obtém a versão exata do código-fonte num ambiente limpo.
2. **Set up Java 21 (`actions/setup-java@v4`):** Configura o JDK 21 (distribuição `temurin`) e ativa o cache automático de dependências do Maven (`cache: maven`).
3. **Make Maven Wrapper executable:** Aplica permissão de execução (`chmod +x mvnw`) ao script do Maven Wrapper versionado no projeto.
4. **Build and run tests:** Executa `./mvnw --batch-mode --no-transfer-progress clean verify`. Esta etapa compila o código, corre os testes automatizados, gera o pacote `.jar` e realiza verificações de integração. Se ocorrer qualquer falha de compilação ou teste, o fluxo é interrompido imediatamente.
5. **Prepare traceable artifact:** Cria o diretório `dist/` contendo:
* O binário gerado (`target/biblioteca-*.jar`).
* O ficheiro `BUILD_INFO.txt` com metadados completos de rastreabilidade (Projeto, Commit SHA, Branch, Run ID, Run Attempt, Data/Hora UTC, versão do Java e comando de build).
* O ficheiro `SHA256SUMS.txt` com o hash criptográfico SHA-256 do `.jar` para validação de integridade e imutabilidade.


6. **Upload application artifact (`actions/upload-artifact@v4`):** Publica o conteúdo da pasta `dist/` no GitHub Actions com retenção de 30 dias (`retention-days: 30`) e validação estrita (`if-no-files-found: error`).

### 3. Dependências

* **Ambiente Executor (Runner):** `ubuntu-latest`
* **Runtime e Build:** Java 21 (Temurin) + Maven Wrapper (`./mvnw`) e dependências declaradas no `pom.xml`
* **GitHub Actions utilizadas:**
* `actions/checkout@v6`
* `actions/setup-java@v4`
* `actions/upload-artifact@v4`



### 4. Artefato Produzido

* **Identificação Única:** `biblioteca-${{ github.run_id }}-${{ github.sha }}`
* **Conteúdo do Pacote (`dist/`):**
* `biblioteca-*.jar` (Aplicação empacotada)
* `BUILD_INFO.txt` (Metadados de rastreabilidade da execução e do commit)
* `SHA256SUMS.txt` (Checksum SHA-256 do binário gerado)



---

## 📸 Evidências de Execução

### ✅ Evidência de Execução Bem-Sucedida

> *Substitua este bloco pelos dados e capturas de ecrã da execução que passou em todas as etapas.*

* **Commit (SHA):** `[Inserir hash do commit, ex: a1b2c3d]`
* **Run ID / Link da Execução:** `[Inserir URL da execução na aba Actions]`
* **Resumo:** Etapas de checkout, configuração do Java 21, `clean verify`, geração de metadados (`BUILD_INFO.txt` e `SHA256SUMS.txt`) e upload do artefato concluídas com sucesso.

*(Insira acima o print da execução verde no GitHub Actions e do artefato `biblioteca-<run_id>-<sha>` gerado)*

---

### ❌ Evidência de uma Falha Detetada

> *Substitua este bloco pelos dados e capturas de ecrã do cenário onde um teste ou código foi alterado propositadamente para falhar.*

* **Commit (SHA):** `[Inserir hash do commit com falha, ex: f9e8d7c]`
* **Run ID / Link da Execução:** `[Inserir URL da execução que falhou]`
* **Causa simulada:** `[Ex: Alteração proposital num teste unitário/integração para simular quebra de regra de negócio]`.
* **Comportamento do Pipeline:** O erro foi detetado na etapa **Build and run tests** (`./mvnw ... clean verify`), interrompendo imediatamente o pipeline e impedindo a execução das etapas **Prepare traceable artifact** e **Upload application artifact**.

*(Insira acima o print mostrando a etapa "Build and run tests" vermelha e as etapas seguintes canceladas/não executadas)*

---

## 🔍 Explicações Técnicas e Operacionais

### 1. Como Reproduzir o Processo

O pipeline foi desenhado para ser 100% reprodutível em qualquer ambiente:

* **No GitHub Actions:** Basta realizar um `push` ou `pull_request` para `main`/`master`, ou aceder à aba **Actions > CI > Run workflow** (`workflow_dispatch`). O uso do `ubuntu-latest`, `setup-java@v4` (Java 21 Temurin) e do **Maven Wrapper (`./mvnw`)** garante que a versão exata do Maven e do JDK seja utilizada sem depender de configurações locais.
* **Em máquina local:** Executando `./mvnw --batch-mode --no-transfer-progress clean verify` com Java 21 e a variável `API_SECURITY_TOKEN_SECRET` definida, obtém-se exatamente o mesmo comportamento determinístico do pipeline.

### 2. Como Investigar Falhas

Caso o pipeline falhe:

1. Aceda à aba **Actions** no repositório do GitHub e selecione a execução marcada com erro (❌).
2. Clique no job **Build, test, and package** para visualizar qual passo falhou.
3. Expanda os logs da etapa **Build and run tests** (onde o comando `./mvnw --batch-mode --no-transfer-progress clean verify` é executado).
4. Procure pelas tags `[ERROR]` no log ou pela secção `Results :` do Maven Surefire/Failsafe para identificar o teste ou classe que quebrou.
5. Caso o erro ocorra na etapa de artefato, a configuração `if-no-files-found: error` sinalizará imediatamente se o `.jar` não tiver sido gerado no caminho esperado.

### 3. Como os Segredos e Permissões são Protegidos

* **Princípio do Menor Privilégio:** O pipeline define explicitamente `permissions: contents: read` no nível global do workflow, garantindo que o `GITHUB_TOKEN` tenha apenas permissão de leitura sobre o repositório, bloqueando modificações não autorizadas no código ou em releases.
* **Isolamento de Variáveis de Ambiente:** A configuração sensível exigida pela aplicação (`API_SECURITY_TOKEN_SECRET`) é injetada via variável de ambiente (`env:`) restrita apenas à etapa de teste (`Build and run tests`), utilizando um valor exclusivo e fictício para o contexto de CI (`ci-test-secret-...`), sem expor segredos reais de ambientes produtivos.
* **Gestão de Segredos Reais:** Para etapas futuras de publicação em registos externos ou deploy (CD), as credenciais reais devem ser armazenadas no cofre **GitHub Actions Secrets** (`Settings > Secrets and variables > Actions`) e referenciadas via `${{ secrets.NOME_DO_SEGREDO }}`, garantindo mascaramento automático nos logs e zero exposição em texto puro no repositório.


