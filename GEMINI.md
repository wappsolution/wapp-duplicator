# gemini.md

## 1. Visão Geral

Este projeto foi criado para **automatizar a duplicação de ambientes** no sistema **uMov.me Center**. Ele simula, via script em Python com Selenium, o fluxo que hoje é feito manualmente:

1. Login no ambiente fonte.
2. Acesso ao menu “Duplicar Ambiente”.
3. Início da duplicação.
4. Espera pela conclusão (~15 minutos).
5. Acesso ao novo ambiente duplicado.
6. Geração da chave de API.
7. Registro do token e status em banco de dados.

### 🧩 Objetivo

Automatizar toda essa sequência para:

- Eliminar cliques repetitivos e risco de erro humano.
- Registrar automaticamente o estado e resultado de cada duplicação.
- Garantir que o token gerado seja salvo de forma segura.

### 📈 Benefícios

- **Ganho de tempo**: substitui ~20 minutos de atividade manual por execução automatizada.  
- **Confiabilidade**: menos erros introduzidos por cliques ou atrasos.  
- **Rastreabilidade**: cada duplicação é registrada no banco com status e token.  

### 🛠️ Tecnologias envolvidas

- **Python 3.x** + **Selenium** (via WebDriver, ex: ChromeDriver)  
- **MySQL** com ORM **Prisma**  
- **pytest** para testes automatizados  
- Estrutura modular (`main.py`, `login.py`, `duplicador.py`, `token_generator.py`, etc.)

---

## 2. Pré‑requisitos ✅

### 📦 Dependências principais  
Instale com:

~~~bash
pip install selenium pytest python-dotenv mysql-connector-python
~~~

- selenium – automação via WebDriver  
- pytest – testes automatizados  
- python-dotenv – leitura de variáveis do `.env`  
- mysql‑connector‑python – registrar pedidos e status no MySQL

### 🌐 WebDriver  
- Baixe e coloque o ChromeDriver (compatível com seu navegador) no PATH ou diretório do projeto.  
  Alternativamente, use GeckoDriver para Firefox.

### 🗃️ Banco de dados MySQL  
- Prepare uma instância acessível de MySQL.  
- Defina em `.env`:

~~~env
DATABASE_URL="mysql://user:password@localhost:3306/duplicator"
~~~

### 🔧 Ambiente de Testes  
- Crie arquivo `pytest.ini` com:

~~~ini
[pytest]
addopts = --maxfail=1 --disable-warnings -q
~~~

- Em `conftest.py`, adicione fixtures para:
  - WebDriver  
  - Leitura do `.env` via `python-dotenv`  
  - Conexão com MySQL usando `mysql-connector-python`

---

## 3. Instalação e Setup ✅

### 3.1 Clonar o repositório
~~~bash
git clone https://github.com/seu-usuario/duplicator-umov.git
cd duplicator-umov
~~~

### 3.2 Criar e ativar ambiente virtual
~~~bash
python3 -m venv venv
source venv/bin/activate   # Linux/macOS
venv\Scripts\activate      # Windows
~~~

### 3.3 Instalar dependências
~~~bash
pip install -r requirements.txt
~~~

**Conteúdo sugerido para `requirements.txt`:**
~~~text
selenium
pytest
python-dotenv
mysql-connector-python
~~~

### 3.4 Configuração inicial
- Copie `.env.example` para `.env` e preencha com:
~~~text
UMOV_URL=https://petroms.umov.me
UMOV_USER=seu_usuario
UMOV_PASSWORD=sua_senha
DATABASE_URL="mysql://user:password@host:port/dbname"
~~~

- Crie diretório para logs e capturas:
~~~bash
mkdir logs
~~~

### 3.5 Executar migração do banco (se aplicável)
~~~bash
python migrations/run_migration.py
~~~

---

## 4. Arquitetura do Projeto ✅

Este projeto segue um design modular, onde cada responsabilidade é isolada em um arquivo, mantendo `main.py` conciso e focado apenas na orquestração das etapas.

### 📦 Componentização

- **main.py**  
  - A função principal que carrega configuração, inicializa os módulos, coordena a execução e trata os fluxos de sucesso ou erro.

- **login.py**  
  - Encapsula a lógica de login no ambiente uMov.me usando Selenium.  
  - Entrada: `username` e `password`.  
  - Saída: instância do `WebDriver` autenticada.

- **duplicador.py**  
  - Contém a lógica para navegar até “Duplicar Ambiente”, clicar, monitorar status e capturar o link do ambiente duplicado.

- **token_generator.py**  
  - Encapsula a geração da chave de API no novo ambiente. Extrai o token e retorna o valor ao chamador.

- **db_adapter.py**  
  - Responsável pela conexão MySQL (via `mysql-connector-python`), CRUD de pedidos e atualizações de status.

- **scheduler.py** *(opcional)*  
  - Componente que permite agendamentos com `cron` ou executores programados para rodar duplicações automaticamente.

### 🔧 Camadas do sistema

- **Core**: módulos principais (`main`, `login`, `duplicador`, `token_generator`)
- **Infra**: Selenium WebDriver + MySQL via `db_adapter`
- **Utility**: logging, configuração (`.env`), tratamento de exceções

### 📈 Motivações e Benefícios

1. **Coesão**: cada módulo faz apenas uma coisa.
2. **Testabilidade**: módulos pequenos facilitam testes unitários e mocks.
3. **Manutenibilidade**: separação clara de responsabilidades.
4. **Escalabilidade**: facilita adição de novos módulos (ex: notifier.py, validator.py).

---

## 5. Estrutura de Diretórios ✅

Estrutura recomendada para o projeto:

~~~text
project_root/
├── src/
│   ├── main.py
│   ├── login.py
│   ├── duplicador.py
│   ├── token_generator.py
│   ├── db_adapter.py
│   ├── scheduler.py
│   └── utils/
│       ├── config.py
│       ├── logger.py
│       └── exceptions.py
├── migrations/
│   └── run_migration.py
├── tests/
│   ├── e2e/
│   │   └── test_flow.py
│   └── unit/
│       ├── test_login.py
│       ├── test_duplicador.py
│       └── test_token_generator.py
├── drivers/
│   ├── chromedriver.exe
│   └── geckodriver.exe
├── logs/
├── requirements.txt
├── pytest.ini
├── conftest.py
└── .env.example
~~~

- **src/**: código principal, modularizado.  
- **utils/**: módulos de configuração, logging e exceções.  
- **migrations/**: scripts de banco.  
- **tests/**: testes automatizados (e2e e unitários).  
- **drivers/**: executáveis do Selenium.  
- **logs/**: logs e screenshots.  
- Raiz: arquivos de configuração e definição do ambiente.

---
## 7. Módulos Principais ✅

### 7.1 login.py

~~~python
# AIDEV-NOTE: módulo de autenticação via Selenium
# AIDEV-GENERATED: exemplo de função de login com Selenium

from selenium.webdriver.remote.webdriver import WebDriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

def login(driver: WebDriver, url: str, username: str, password: str) -> None:
    """
    Acessa a URL de login, preenche as credenciais e submete o formulário.
    Lança TimeoutException se os elementos não ficarem visíveis.
    """
    driver.get(url)
    
    # Espera pelo campo de usuário, preenche e faz o mesmo para a senha
    user_field = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.ID, "username-input"))
    )
    user_field.send_keys(username)
    
    pass_field = driver.find_element(By.ID, "password-input")
    pass_field.send_keys(password)
    
    # Clica no botão de login
    login_button = driver.find_element(By.XPATH, "//button[@type='submit']")
    login_button.click()
    
    # Espera por um elemento pós-login para confirmar sucesso
    WebDriverWait(driver, 15).until(
        EC.presence_of_element_located((By.ID, "dashboard-welcome-message"))
    )
~~~

### 7.2 duplicador.py

~~~python
# AIDEV-NOTE: módulo para acionar duplicação e extrair link
# AIDEV-GENERATED: exemplo de função para duplicar ambiente

from selenium.webdriver.remote.webdriver import WebDriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

def duplicar_ambiente(driver: WebDriver) -> str:
    """
    Navega até a função de duplicação, inicia o processo e
    aguarda a conclusão para retornar o link do novo ambiente.
    """
    # Navega até o menu de administração e depois para a duplicação
    admin_menu = WebDriverWait(driver, 10).until(
        EC.element_to_be_clickable((By.ID, "admin-menu-item"))
    )
    admin_menu.click()
    
    duplicate_link = WebDriverWait(driver, 10).until(
        EC.element_to_be_clickable((By.LINK_TEXT, "Duplicar Ambiente"))
    )
    duplicate_link.click()

    # Aciona o botão que inicia a duplicação
    start_button = driver.find_element(By.CSS_SELECTOR, "button.start-duplication")
    start_button.click()

    # Aguarda a mensagem de conclusão (timeout longo, conforme especificado)
    completion_element = WebDriverWait(driver, 900).until(
        EC.presence_of_element_located((By.XPATH, "//*[contains(text(), 'Ambiente duplicado com sucesso')]"))
    )
    
    # Extrai e retorna o link do novo ambiente
    new_env_link = completion_element.find_element(By.TAG_NAME, "a").get_attribute("href")
    return new_env_link
~~~

### 7.3 token_generator.py

~~~python
# AIDEV-NOTE: módulo para geração e captura do token
# AIDEV-GENERATED: exemplo de função para gerar e extrair token de API

from selenium.webdriver.remote.webdriver import WebDriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

def gerar_token(driver: WebDriver) -> str:
    """
    Acessa a área de API, clica para gerar um novo token e
    retorna o valor gerado.
    """
    # Navega até a seção de API
    api_menu_item = WebDriverWait(driver, 15).until(
        EC.element_to_be_clickable((By.ID, "api-settings-link"))
    )
    api_menu_item.click()

    # Clica no botão para gerar um novo token
    generate_button = WebDriverWait(driver, 10).until(
        EC.element_to_be_clickable((By.CSS_SELECTOR, "button#generate-token"))
    )
    generate_button.click()

    # Espera o campo do token aparecer e extrai seu valor
    token_element = WebDriverWait(driver, 20).until(
        EC.visibility_of_element_located((By.CSS_SELECTOR, "span.api-token-value"))
    )
    
    api_token = token_element.text
    return api_token
~~~
~~~markdown
## 8. Fluxo de Execução ✅

- main.py inicia o processo (manual ou via scheduler)
- Conexão com banco (db_adapter) é estabelecida
- Pedido de duplicação é criado com status PENDING
- login.py autentica no ambiente fonte via Selenium
- duplicador.py dispara duplicação e aguarda conclusão
   - Em caso de erro ou timeout, atualiza status para FAILED, registra e encerra
- Se suceder, recebe o link do novo ambiente
- token_generator.py acessa novo ambiente e gera o token da API
- Token é salvo no banco, status atualizado para COMPLETED
- O processo finaliza com log de sucesso ou erro
~~~

~~~markdown
## 9. Especificações de Espera e Timeouts ✅

- **Login** (`login.py`):  
  - Espera até 20 segundos por campos de usuário e senha.
  - Timeout padrão de login ajustável via variável de ambiente `LOGIN_TIMEOUT_SEC`.

- **Duplicação** (`duplicador.py`):  
  - Ação de duplicar aguarda até 900 segundos (15 minutos) pelo status "Pronto".
  - Configuração de retry está incluída:  
    - Tenta até 3 vezes com espera exponencial (1, 2, 4 segundos) em caso de falhas temporárias.  
    - Timeout geral ajustável via `DUPLICATION_TIMEOUT_SEC`.

- **Geração de token** (`token_generator.py`):  
  - Espera até 20 segundos pelo botão e pelo valor do token.
  - Timeout configurável via `TOKEN_TIMEOUT_SEC`.

- **Tempo entre requisições ao banco**:  
  - Após criar ou atualizar um registro, espera 0.5 segundos para evitar sobrecarga do banco.

### Variáveis de ambiente relacionadas:
- `LOGIN_TIMEOUT_SEC`  
- `DUPLICATION_TIMEOUT_SEC`  
- `TOKEN_TIMEOUT_SEC`  
- `DB_WRITE_PAUSE_SEC`  

### Comportamentos em caso de timeout:
- Cada função deve lançar uma `TimeoutException`.
- `main.py` captura a exceção, atualiza o status para `FAILED`, e encerra o processo com log.

~~~  
~~~markdown
## 10. Captura e Armazenamento de Token ✅

- **token_generator.py** acessa o ambiente clonado e gera o token via interface:
  - Clica no menu/API.
  - Gera o token.
  - Extrai o valor com `WebDriverWait`.

- **Persistência no banco** (`db_adapter.py`):
  - Recebe `(request_id, token)`.
  - Executa `UPDATE DuplicationRequest SET apiToken = ?, status = 'COMPLETED', completionTime = NOW() WHERE id = ?`.
  - Em caso de erro, faz rollback e lança exceção.

- **Boas práticas**:
  - **Proteção do token**: não logar o token completo em ambientes públicos.
  - **Encriptação opcional**: salvar usando variável `ENCRYPT_TOKENS=true/false`.
  - **Limpeza pós-processo**: se `ENCRYPT_TOKENS=false`, sobrescrever o valor plaintext após uso.

- **Confirmação no console**:
  - Exibe linha resumida:  
    `Duplicacao [id]: token salvo com sucesso.`

- **Exemplo de chamada no `main.py`**:
  ```python
  token = token_generator.gerar_token(driver_clonado)
  db_adapter.save_token(request_id, token)
```
- **Erros possíveis:**
- TimeoutException se token não for visível.
- Falha no DB lança DatabaseError, que é capturado por main.py para marcar FAILED.

~~~markdown
## 11. Logging e Tratamento de Erros ✅

### 🔍 Logging
- Utiliza módulo `logger.py` com configurações:
  - Log em nível `INFO` e `DEBUG`.
  - Saída para console e arquivo: `logs/duplicator.log`.
  - Formato padronizado com timestamp, módulo e mensagem.
- Captura de screenshots no erro:
  ```python
  driver.save_screenshot(f"logs/error_{request_id}.png")
Exemplo de chamada no main.py:

python

logger.info("Iniciando duplicação", extra={"request_id": request_id})
🛡️ Tratamento de Erros
Uso de bloco try/except no main.py:

Captura TimeoutException, DatabaseError, WebDriverException.

Atualiza status para FAILED no banco.

Realiza driver.save_screenshot(...).

Loga o erro com logger.error(...).

Em caso de falha, o processo encerra com código de saída diferente de zero.

Retry:

Para erros temporários (ex: falha ao clicar botão), tenta até 2 vezes antes de falhar definitivamente.

Intervalo exponencial entre tentativas.

🔁 Fluxo de tratamento no main.py
Tenta login → falha? registra FAILED, screenshot, encerra.

Tenta duplicar → falha? atualiza FAILED, screenshot, encerra.

Tenta gerar token → falha? atualiza FAILED, screenshot, encerra.

Ao final, em caso de sucesso ou falha, escreve um log final:
Duplicação [id] finalizada com status: XXX.


## 12. Testes Automatizados ✅

### Estrutura dos testes
- Testes separados em `tests/unit/` e `tests/e2e/`  
- Padrão **Page Object Model** para os testes E2E com Selenium

### pytest.ini
```ini
[pytest]
addopts = --maxfail=1 --disable-warnings -q
```

### Fixtures (`conftest.py`)
- `driver`: instanciado com ChromeDriver ou GeckoDriver (modo headless)
- `env_config`: carrega URL, usuário e senha via `python-dotenv`
- `db`: conexão com MySQL via `mysql-connector-python`

### Exemplo – Teste Unitário
```python
def test_login_success(monkeypatch, driver, env_config):
    assert login.login(driver, env_config.url, env_config.user, env_config.password) is None
```

### Exemplo – Teste E2E (fluxo completo)
```python
def test_full_flow(driver, env_config, db):
    request_id = db.create_request(env_config.source_env)
    login.login(driver, env_config.url, env_config.user, env_config.password)
    link = duplicador.duplicar_ambiente(driver)
    driver.get(link)
    token = token_generator.gerar_token(driver)
    db.save_token(request_id, token)
    assert db.get_status(request_id) == "COMPLETED"
```

### Coverage e CI
- Comando: `pytest --cov=src`  
- Meta de cobertura mínima: ≥ 80%  
- Integração com CI: GitHub Actions, GitLab CI etc.

## 13. Planejamento de Sprints ✅

### ✅ To-do por Sprint

 **Sprint 1 – Setup Inicial e Estrutura do Projeto**
- [ ] Clonagem e estruturação inicial do repositório (`src/`, `tests/`, `drivers/`, `logs/`, etc.)
- [ ] Criação de ambiente virtual e `requirements.txt`
- [ ] Configuração de `.env.example` com variáveis básicas
- [ ] Configuração inicial do MySQL e teste de conexão
- [ ] Configuração de logging básico (arquivo + console)
- [ ] Criação de `main.py` com estrutura de orquestração
- [ ] Setup do `pytest`, `pytest.ini` e `conftest.py`

 **Sprint 2 – Implementação da Automação com Selenium**
- [ ] Implementação do módulo `login.py` com autenticação via Selenium
- [ ] Implementação de `duplicador.py` com espera por conclusão (~15 min)
- [ ] Criação de `db_adapter.py` com conexão e CRUD básico
- [ ] Criação de `token_generator.py` com extração da chave via WebDriver
- [ ] Integração dos módulos no `main.py` com fluxo linear completo
- [ ] Configuração de timeouts e retries via `.env`

 **Sprint 3 – Testes Automatizados e Tratamento de Erros**
- [ ] Criação de testes unitários para `login`, `duplicador` e `token_generator`
- [ ] Criação de teste E2E com fluxo completo e banco
- [ ] Captura de screenshots em falhas
- [ ] Logging estruturado (request_id, erro, status, etc.)
- [ ] Tratamento robusto de exceções (`TimeoutException`, `DatabaseError`, etc.)
- [ ] Teste e validação do retry com backoff exponencial

 **Sprint 4 – Refatoração, CLI e Agendamento**
- [ ] Refatoração dos módulos para manter <100 linhas por arquivo
- [ ] Criação do `scheduler.py` com integração via `schedule` ou cron
- [ ] Adição de flags `--once` e `--verbose` no `main.py`
- [ ] Testes de execução agendada via cron (`crontab -e`)
- [ ] Separação de responsabilidades em `utils/` (`config.py`, `logger.py`, `exceptions.py`)
- [ ] Documentação CLI e instruções para cron e `xvfb-run`

 **Sprint 5 – Segurança, Extensões e Finalização**
- [ ] Implementar criptografia de token (condicional via `ENCRYPT_TOKENS`)
- [ ] Implementar logs de duplicações com status e duração no banco
- [ ] Adicionar suporte para múltiplos browsers via `.env`
- [ ] Criar alertas por email ou Slack em falhas de duplicação
- [ ] Melhorar documentação (`gemini.md`, README, CHANGELOG)
- [ ] Realizar teste final com cenário completo de duplicação real

## 14. Boas Práticas e Padrões ✅

### 📐 Modularização clara  
- Cada arquivo (`login.py`, `duplicador.py`, etc.) tem **no máximo ~100 linhas**  
- **main.py** serve apenas como orquestrador; lógica pesada é delegada a módulos  
- Respeito ao princípio **Single Responsibility** em cada módulo

### 🚀 Qualidade de código  
- Use **typing** em funções públicas (ex: `def login(...) -> None:`)  
- Aplique **linters** (flake8 ou pylint) e **formatadores** (black)  
- Documente funções com **docstrings** no estilo Google ou NumPy  

### 🔄 Robustez e manutenibilidade  
- Use padrões **retry/exponencial backoff** para falhas temporárias  
- **Captura de screenshots** e logs detalhados ao ocorrer erros  
- **Limites de timeout** configuráveis via `.env` ou `config.py`  

### 🔐 Segurança  
- Tokens e credenciais não devem ser colocados no código  
- Armazenamento seguro: variáveis de ambiente, `.env`, ou criptografia  
- Remova valores sensíveis dos logs (mas registre que ação foi concluída)

### 🤝 Consistência entre ambientes  
- Gere estruturas idênticas em ambientes de desenvolvimento e produção  
- Use `chromedriver` compatível com sua versão do Chrome/driver  
- Mantenha scripts de migração do banco no controle de versão

### 📚 Documentação e rastreabilidade  
- Inclua comentários **AIDEV-** para marcar geração/ajustes por IA  
- Tenha documentação e o `gemini.md` sempre atualizados  
- Corte de tópicos spam: mantenha code comments e documentação sincronizados  

---

## 15. Execução e Automação (CLI / Cron) ✅

### 🔁 Execução Manual (CLI)
- Execute diretamente com:
python src/main.py

yaml
Copiar
Editar
- Parâmetros opcionais:
- `--once`: executa apenas uma duplicação
- `--verbose`: ativa logs em nível DEBUG
- Saída padrão exibe `request_id`, status e link do ambiente criado

---

### 📆 Agendamento com Cron (Linux/macOS)
1. Edite o cron:
crontab -e

markdown
Copiar
Editar
2. Exemplo de entrada para executar todo dia às 2h:
0 2 * * * cd /caminho/para/project_root && /usr/bin/python3 src/main.py --once >> logs/cron.log 2>&1

markdown
Copiar
Editar
3. Se usar navegador não-headless, configure o `DISPLAY`, por exemplo:
DISPLAY=:0

perl
Copiar
Editar
ou use `xvfb-run`:
0 2 * * * cd /… && xvfb-run /usr/bin/python3 src/main.py --once >> logs/cron.log 2>&1

yaml
Copiar
Editar
Isso garante execução sem interface gráfica :contentReference[oaicite:0]{index=0}.

---

### 🐍 Alternativa: Python `schedule` (turbinado)
- Útil se não quiser usar cron diretamente.
- Exemplo:
```python
import schedule, time
schedule.every().day.at("02:00").do(lambda: os.system("python src/main.py --once"))
while True:
    schedule.run_pending()
    time.sleep(60)
Ajuste conforme carga de trabalho .

🛠️ Boas práticas de agendamento
Aspecto	Recomendações
PATH e ambiente	Use caminhos absolutos para Python e drivers
Logs	Capture stdout e stderr (>> cron.log 2>&1)
Display	Configure DISPLAY ou use xvfb-run
Monitoramento	Verifique logs/cron.log e logs/duplicator.log

## 16. Sugestões de Extensão ✅

### 1. Integração com browser extensions  
- Carregar extensões no ChromeDriver para testes avançados ou acesso a funcionalidades específicas :contentReference[oaicite:0]{index=0}.

### 2. Integração UI + API  
- Adicione testes que validem tanto a UI com Selenium quanto chamadas diretas à API (via Requests), possibilitando cobertura total do fluxo :contentReference[oaicite:1]{index=1}.

### 3. Suporte a múltiplos browsers  
- Use `chromedriver`, `geckodriver`, e evite hardcode: permita configuração via `.env` para Chrome ou Firefox :contentReference[oaicite:2]{index=2}.

### 4. Paralelização de testes  
- Integre com **Selenium Grid** ou frameworks como Robot Framework/Pytest-xdist para executar testes concorrentes :contentReference[oaicite:3]{index=3}.

### 5. Gerador de código automático  
- Utilize extensões como “Selenium Auto Code Generator” para acelerar criação de scripts de automação :contentReference[oaicite:4]{index=4}.

### 6. Monitoramento e notificações  
- Envie alertas por e-mail ou Slack quando duplicações falharem ou completarem.  
- Adicione métricas básicas (duracao, taxas de sucesso).

### 7. Agendamento avançado e dashboards  
- Crie painel visual para monitorar status de duplicações e history.  
- Implemente UI com Flask ou FastAPI + gráfico simples.

---
## ✅ Conclusão Final do `gemini.md`

Com todos os blocos implementados, este documento `gemini.md` serve como:

- 📚 Guia técnico completo para desenvolvimento, execução e manutenção
- 🛠️ Referência modular clara, com responsabilidades separadas por arquivo
- 🔐 Registro de segurança, logs e tratamento de falhas
- 🚀 Base sólida para testes, CI/CD, automação e futuras extensões
- 📅 Planejamento de sprints com rastreabilidade técnica
- 📌 Alinhamento com boas práticas e rastreabilidade de código gerado por IA

---

### 🧠 Acompanhamento e Sugestão de Manutenção

- Atualize este documento sempre que:
  - Novos módulos forem criados
  - Mudanças no fluxo principal ocorrerem
  - Campos ou formatos de banco forem alterados
  - Sprints mudarem de escopo
- Utilize sempre os comentários especiais para rastrear trechos gerados por IA:
  - `# AIDEV-NOTE:`
  - `# AIDEV-TODO:`
  - `# AIDEV-QUESTION:`
  - `# AIDEV-GENERATED:`

---



