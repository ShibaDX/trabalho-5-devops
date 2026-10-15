
# Trabalho 5 (T5) — Pipeline CI Mínimo e Reprodutível

**Disciplina:** Cloud Computing e DevOps Avançado  
**Curso:** Sistemas para Internet | 6º Período  
**Professor:** Diogo Ranghetti  

---

## 👥 Integrantes do Grupo

- **[Gabriel Fonseca Proença]** - [RA: 14608]
- **[Guilherme Carrera]** - [RA: 14651]
- **[Kaiky Massaki Ishibashi]** - [RA: 14535]
- **[Marcos Kaiky Rodrigues Garcia]** - [RA: 14610]

---

## 📌 Sobre o Projeto

Este repositório contém a implementação de um pipeline de **Integração Contínua (CI)** mínimo, automatizado e reprodutível para a aplicação **[Biblioteca]**. O objetivo é garantir que cada alteração no código seja verificada automaticamente através de compilação, execução de testes e geração de um artefato rastreável a partir de um ambiente limpo.

- **Ficheiro de definição do pipeline:** `[.github/workflows/main.yml]`

---

## 🚀 Instruções de Execução

### Execução Automática (via CI)
1. Realize uma alteração no código-fonte num ramo de desenvolvimento ou diretamente na branch configurada.
2. Faça o commit das alterações:

   ```
   git add .
   git commit -m "feat: descrição da alteração"
   ```

3. Envie as alterações para o repositório remoto (`git push origin main`) ou abra um **Pull Request** apontando para a branch `main`.
4. Vá para o separador **[Actions / CI-CD]** do repositório para acompanhar a execução automática em tempo real.

### Execução Local (Ambiente de Desenvolvimento)

Caso deseje validar o build e os testes na sua máquina antes de enviar para o repositório:

1. Clone este repositório:
   
```
git clone https://github.com/ShibaDX/trabalho-5-devops.git
cd ./trabalho-5-devops/
```

2. Certifique-se de ter o **Java [Versão, ex.: 21]** e o **Maven** instalados.
   
3. Execute a compilação e os testes em modo batch:

```
mvn --batch-mode clean test
```


4. Gere o pacote da aplicação (artefato `.jar`):
```
mvn --batch-mode package
```



---

## ⚙️ Estrutura e Identificação do Pipeline

### 1. Gatilho (Trigger)

* **Eventos que iniciam o pipeline:**
* `push` na branch `main`.
* Abertura ou atualização de `pull_request` direcionado à branch `main`.
* `[Opcional: workflow_dispatch para execução manual autorizada]`.



### 2. Etapas (Steps / Stages)

O fluxo do pipeline está dividido nas seguintes etapas sequenciais:

| Ordem | Etapa | Comando / Ação Executada | Condição de Passagem |
| --- | --- | --- | --- |
| **1** | **Checkout** | `actions/checkout@v4` | Obter a versão exata do código do commit atual num ambiente limpo. |
| **2** | **Setup do Ambiente** | `actions/setup-java@v4` | Configurar o JDK `[ex.: 17]` e ativar o cache de dependências do Maven. |
| **3** | **Build e Testes** | `mvn --batch-mode clean test` | O código deve compilar sem erros e 100% dos testes unitários/integração devem passar. |
| **4** | **Empacotamento** | `mvn --batch-mode package -DskipTests` | Gerar o ficheiro `.jar` na diretoria `target/` apenas se a etapa anterior tiver sucesso. |
| **5** | **Publicação do Artefato** | `actions/upload-artifact@v4` | Armazenar o artefato gerado associando-o ao hash (`SHA`) do commit. |

### 3. Dependências

* **Ambiente de Execução (Runner):** `ubuntu-latest` (ambiente limpo provisionado a cada execução).
* **Runtime e Ferramentas:**
* Java JDK `[Versão, ex.: 21 ( distribuição Eclipse Temurin )]`.
* Apache Maven `[Versão padrão do runner ou Maven Wrapper ./mvnw]`.
* Docker `[Caso construam imagem de contentor no pipeline]`.


* **Dependências da Aplicação:** Geridas via ficheiro `pom.xml` (transferidas automaticamente a partir do Maven Central durante a execução).

### 4. Artefato Produzido

* **Tipo de Artefato:** Pacote Java executável (`.jar`) `[e/ou Imagem Docker]`.
* **Caminho de Geração:** `target/[nome-do-projeto]-[versao].jar`.
* **Identificação e Rastreabilidade:** O artefato é publicado no pipeline com o formato `app-jar-<COMMIT_SHA>` (ex.: `app-jar-a1b2c3d`), garantindo que cada pacote gerado está inequivocamente vinculado ao commit exato que o originou.

---

## 📸 Evidências de Execução

### ✅ 1. Evidência de Execução Bem-Sucedida

> **Contexto:** Execução realizada a partir do commit `[inserir hash do commit, ex.: a1b2c3d]`, onde a compilação, todos os testes automatizados e o empacotamento ocorreram sem erros.

* **Link direto para a execução (Run):** `[Inserir URL da execução no GitHub Actions / GitLab]`
* **Registo / Captura de Ecrã:**


*(Substitua a linha acima pelo print das etapas verdes no pipeline e do artefato gerado disponível para transferência)*

```
[Opcional: Cole aqui o trecho final do log do Maven comprovando os testes executados e o BUILD SUCCESS]
Exemplo:
Tests run: 12, Failures: 0, Errors: 0, Skipped: 0
[INFO] Building jar: /home/runner/work/.../target/biblioteca-0.0.1-SNAPSHOT.jar
[INFO] BUILD SUCCESS

```

---

### ❌ 2. Evidência de Falha Detetada

> **Contexto:** Simulação de erro realizada no commit `[inserir hash do commit de falha]`, através da alteração propositada de `[ex.: uma asserção no teste de cálculo de empréstimo / erro de compilação]`. O pipeline detetou o problema na etapa de **Testes** e interrompeu imediatamente o fluxo, impedindo o empacotamento e a publicação de um artefato defeituoso.

* **Link direto para a execução com falha (Run):** `[Inserir URL da execução que falhou]`
* **Registo / Captura de Ecrã:**


*(Substitua a linha acima pelo print mostrando a etapa de testes vermelha e as etapas seguintes canceladas/ignoradas)*

```
[Opcional: Cole aqui o trecho do log que mostra a falha exata apontada pelo pipeline]
Exemplo:
[ERROR] Tests run: 12, Failures: 1, Errors: 0, Skipped: 0
[ERROR] CalcEmprestimoTest.deveCalcularPrazoCorretamente:42 expected: <7> but was: <10>
[INFO] BUILD FAILURE
Error: Process completed with exit code 1.

```

---

## 🔍 Explicações Técnicas e Operacionais

### 1. Como Reproduzir o Processo

O pipeline foi desenhado para ser **100% independente da máquina local** de qualquer programador:

* Todas as instruções do fluxo estão versionadas como código no ficheiro `[.github/workflows/ci.yml]`.
* Para outra equipa reproduzir o processo, basta realizar um *fork* ou clonar este repositório num ambiente com GitHub Actions ativo.
* Como o pipeline utiliza imagens oficiais de runners limpos (`ubuntu-latest`), instala explicitamente a versão fixa do Java e resolve as dependências declaradas no `pom.xml` em modo não interativo (`--batch-mode`), qualquer execução sob o mesmo commit produzirá um resultado consistente e determinístico.

### 2. Como Investigar Falhas

Quando uma execução do pipeline falha, o diagnóstico deve seguir os passos abaixo:

1. **Identificar a Execução e o Commit:** No separador **Actions** (ou diretamente no *check* vermelho do Pull Request/Commit), clicar na execução que apresentou erro.
2. **Localizar a Etapa que Falhou:** O painel visual indicará exatamente em qual passo o fluxo parou (ex.: *Checkout*, *Build* ou *Testes*).
3. **Analisar os Logs de Execução:** Expandir a secção com o ícone de erro (`❌`) para ler a saída do terminal:
* **Falha de Compilação:** Verificar erros de sintaxe ou dependências em falta apontados pelo compilador (`COMPILATION ERROR`).
* **Falha nos Testes:** Localizar o resumo do *Maven Surefire Plugin* para identificar a classe, o método de teste e a linha exata onde o comportamento divergiu do esperado (`Failures` ou `Errors`).


4. **Correção e Revalidação:** Reproduzir o teste localmente (`mvn test`), corrigir o defeito e submeter um novo commit para que o pipeline valide a correção automaticamente.

### 3. Como os Segredos são Protegidos

Para garantir a segurança básica do pipeline e da aplicação, foram adotadas as seguintes práticas:

* **Zero Segredos no Código:** Nenhuma senha, token de API, chave privada ou credencial de banco de dados / registo Docker é armazenada em texto puro no código-fonte, em ficheiros `application.properties` públicos ou no ficheiro YAML do pipeline.
* **Uso de Cofre Nativo da Plataforma:** Todas as variáveis sensíveis necessárias para o pipeline são cadastradas exclusivamente nos **Encrypted Secrets** do repositório (`Settings > Secrets and variables > Actions`) e injetadas apenas em tempo de execução através da sintaxe `${{ secrets.NOME_DO_SEGREDO }}`.
* **Mascaramento em Logs:** Os valores provenientes do cofre de segredos são automaticamente mascarados (`***`) nos logs de execução da plataforma, evitando exposição acidental durante auditorias ou diagnósticos de erro.
* **Princípio do Menor Privilégio:** As permissões do token de execução do pipeline (`GITHUB_TOKEN`) são mantidas restritas apenas ao necessário para a leitura do código e upload de artefatos da execução.

```
