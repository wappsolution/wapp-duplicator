# 🚀 wapp-duplicator

Automação completa do processo de **duplicação de ambientes no uMov.me Center** com geração de token de API e registro no banco de dados.

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![Status](https://img.shields.io/badge/status-em%20desenvolvimento-yellow)

---

## 📦 Visão Geral

Este projeto automatiza, via **Python + Selenium**, o fluxo de duplicação de ambientes que normalmente é realizado manualmente dentro do uMov.me Center.

### ✅ Fluxo automatizado:
1. Login no ambiente origem
2. Acesso à função "Duplicar Ambiente"
3. Acompanhamento do progresso (~15min)
4. Acesso ao novo ambiente criado
5. Geração de token de API
6. Registro de status e token em banco MySQL

---

## 🎯 Objetivo

- Eliminar cliques manuais repetitivos
- Evitar erros humanos no processo
- Garantir registro seguro e rastreável do token
- Suporte a agendamento e execução automática

---

## ⚙️ Tecnologias Utilizadas

| Tecnologia         | Descrição                         |
|--------------------|-----------------------------------|
| Python 3.10+       | Linguagem principal                |
| Selenium WebDriver | Automação de interface gráfica    |
| MySQL              | Persistência dos registros         |
| python-dotenv      | Leitura de variáveis de ambiente  |
| pytest             | Testes automatizados               |

---

## 📁 Estrutura do Projeto

```text
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
├── tests/
├── logs/
├── drivers/
├── .env.example
├── requirements.txt
└── README.md
🛠️ Como Usar
1. Clone o repositório
bash
Copiar
Editar
git clone https://github.com/wappsolution/wapp-duplicator.git
cd wapp-duplicator
2. Crie e ative o ambiente virtual
bash
Copiar
Editar
python -m venv venv
venv\Scripts\activate        # Windows
source venv/bin/activate    # macOS/Linux
3. Instale as dependências
bash
Copiar
Editar
pip install -r requirements.txt
4. Configure o .env
Copie o arquivo .env.example:

bash
Copiar
Editar
cp .env.example .env
Preencha com suas credenciais e parâmetros.

🧪 Executando os Testes
bash
Copiar
Editar
pytest tests/unit
pytest tests/e2e
Para ver cobertura de testes:

bash
Copiar
Editar
pytest --cov=src --cov-report=term-missing
📆 Agendamento com Cron (Linux/macOS)
bash
Copiar
Editar
0 2 * * * cd /caminho/projeto && /usr/bin/python3 src/main.py --once >> logs/cron.log 2>&1
🔒 Segurança
Tokens são tratados com confidencialidade

Armazenamento pode ser criptografado com ENCRYPT_TOKENS=true

.env nunca deve ser versionado

🧩 Extensões Futuras
Dashboard com histórico de duplicações

Notificações por e-mail ou Slack

UI simples para agendamento com Flask/FastAPI

Suporte a múltiplos navegadores

🤝 Contribuindo
Contribuições são bem-vindas!
Para contribuir:

Fork este repositório

Crie sua branch: git checkout -b minha-feature

Commit suas mudanças: git commit -m 'feat: minha nova feature'

Push para sua branch: git push origin minha-feature

Abra um Pull Request ✨

📄 Licença
Este projeto está licenciado sob os termos da MIT License.

👨‍💻 Desenvolvido por
WAPP Solutions
https://github.com/wappsolution
