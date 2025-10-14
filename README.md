# Django Project - Deploy com Coolify

Este é um projeto Django configurado para deploy automático usando [Coolify](https://coolify.io/), baseado no tutorial de [Francisco Macedo](https://fmacedo.com/posts/coolify-your-django-project/).

## 📋 Sobre o Projeto

Este projeto Django demonstra como configurar e fazer deploy de uma aplicação web usando:

- **Django** - Framework web Python
- **WhiteNoise** - Servir arquivos estáticos
- **python-decouple** - Gerenciamento de variáveis de ambiente
- **PostgreSQL** - Banco de dados (com fallback para SQLite)
- **Gunicorn** - Servidor web WSGI
- **Docker** - Containerização
- **Coolify** - Plataforma de deploy

## 🚀 Deploy Rápido

### Pré-requisitos

1. **Servidor VPS ou VM** (recomendado: [Hetzner](https://www.hetzner.com/) a partir de $5/mês)
2. **Coolify instalado** - Siga as [instruções de instalação](https://coolify.io/docs/installation)
3. **Repositório Git** (GitHub, GitLab, Bitbucket, etc.)

### Configuração Local

1. **Clone o repositório:**
```bash
git clone https://github.com/joaopaulosantiago/djproject.git
cd djproject
```

2. **Crie e ative o ambiente virtual:**
```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/Mac
source .venv/bin/activate
```

3. **Instale as dependências:**
```bash
pip install -r requirements.txt
```

4. **Configure as variáveis de ambiente locais (arquivo `.env`):**
```env
SECRET_KEY=django-insecure-sua-chave-aqui
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
CSRF_TRUSTED_ORIGINS=http://localhost:8000,http://127.0.0.1:8000
DATABASE_URL=sqlite:///db.sqlite3
```

5. **Execute as migrações e rode o servidor:**
```bash
python manage.py migrate
python manage.py runserver
```

## 🐳 Deploy com Coolify

### 1. Push para Git

```bash
git add .
git commit -m "Configuração inicial para Coolify"
git push origin main
```

### 2. Configurar no Coolify

1. **Acesse seu painel Coolify** em `http://seu-ip:8000`
2. **Crie um novo projeto**
3. **Adicione um banco PostgreSQL** (opcional - o projeto funciona com SQLite)
4. **Conecte o repositório Git**

### 3. Configurar Variáveis de Ambiente

No painel do Coolify, configure estas variáveis:

```env
SECRET_KEY=sua-chave-secreta-gerada
DEBUG=False
ALLOWED_HOSTS=seu-dominio-coolify.com
CSRF_TRUSTED_ORIGINS=https://seu-dominio-coolify.com
DATABASE_URL=postgres://usuario:senha@host:5432/database
```

**💡 Para gerar uma nova SECRET_KEY:**
```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

### 4. Configurações Avançadas

- ✅ Marque **"Connect To Predefined Network"** se estiver usando PostgreSQL
- ✅ Configure **"Build Pack"** como **Docker Compose**

### 5. Deploy

Clique em **"Deploy"** no Coolify. O deploy será automático a cada push na branch `main`.

## 📁 Estrutura do Projeto

```
djproject/
├── djproject/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py          # Configurações com variáveis de ambiente
│   ├── urls.py              # URLs do projeto
│   ├── views.py             # Views do projeto
│   └── wsgi.py
├── templates/
│   └── index.html           # Template da página inicial
├── Dockerfile               # Configuração Docker
├── docker-compose.yml       # Configuração Docker Compose
├── entrypoint.sh            # Script de inicialização
├── requirements.txt         # Dependências Python
├── manage.py
├── db.sqlite3              # Banco SQLite (desenvolvimento)
└── .gitignore
```

## 🔧 Configurações Importantes

### settings.py

O arquivo `settings.py` está configurado para usar variáveis de ambiente:

```python
from decouple import config
import dj_database_url

# Segurança
SECRET_KEY = config('SECRET_KEY', default='chave-padrao-insegura')
DEBUG = config('DEBUG', default=True, cast=bool)

# Hosts permitidos
ALLOWED_HOSTS = config('ALLOWED_HOSTS', default='localhost,127.0.0.1', 
                      cast=lambda x: [i.strip() for i in x.split(',')])

# CSRF Origins
CSRF_TRUSTED_ORIGINS = config('CSRF_TRUSTED_ORIGINS', 
                             default='http://localhost:8000,http://127.0.0.1:8000',
                             cast=lambda x: [i.strip() for i in x.split(',')])

# Banco de dados
DATABASES = {
    "default": dj_database_url.config(
        default=config("DATABASE_URL", f"sqlite:///{BASE_DIR}/db.sqlite3"),
        conn_max_age=600,
        ssl_require=False,
    ),
}
```

### Docker

O projeto inclui:

- **Dockerfile**: Configuração da imagem Docker
- **docker-compose.yml**: Orquestração dos serviços
- **entrypoint.sh**: Script de inicialização que:
  - Aguarda o volume do servidor
  - Executa migrações do banco
  - Inicia o Gunicorn

## � Copiando o Projeto (Mudando Nome)

Se você copiar este projeto e quiser usar um nome diferente, **OBRIGATORIAMENTE** altere estas configurações:

### 1. **Renomeie a pasta principal**
```bash
# De: djproject/
# Para: seu_novo_nome/
```

### 2. **Arquivo `settings.py`**
Altere a linha `ROOT_URLCONF`:
```python
# De:
ROOT_URLCONF = 'djproject.urls'

# Para:
ROOT_URLCONF = 'seu_novo_nome.urls'
```

Altere a linha `WSGI_APPLICATION`:
```python
# De:
WSGI_APPLICATION = 'djproject.wsgi.application'

# Para:
WSGI_APPLICATION = 'seu_novo_nome.wsgi.application'
```

### 3. **Arquivo `wsgi.py`**
```python
# De:
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'djproject.settings')

# Para:
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'seu_novo_nome.settings')
```

### 4. **Arquivo `asgi.py`**
```python
# De:
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'djproject.settings')

# Para:
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'seu_novo_nome.settings')
```

### 5. **Arquivo `manage.py`**
```python
# De:
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'djproject.settings')

# Para:
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'seu_novo_nome.settings')
```

### 6. **Arquivo `entrypoint.sh`**
```bash
# De:
gunicorn --bind :8000 --workers 2 djproject.wsgi

# Para:
gunicorn --bind :8000 --workers 2 seu_novo_nome.wsgi
```

### 7. **Arquivo `docker-compose.yml`**
```yaml
# Altere o container_name se desejar:
services:
    server:
        container_name: seu_novo_nome  # De: djproject
```

### ⚠️ **ERRO COMUM**: `ModuleNotFoundError: No module named 'djproject'`

Se você esquecer de alterar essas configurações, receberá este erro. Certifique-se de alterar **TODOS** os arquivos listados acima.

## 🔧 Desenvolvimento

### Comandos Úteis

```bash
# Criar superusuário
python manage.py createsuperuser

# Executar migrações
python manage.py migrate

# Coletar arquivos estáticos
python manage.py collectstatic

# Executar testes
python manage.py test
```

### Logs no Coolify

Para debugar problemas:

1. Acesse a aba **"Logs"** no Coolify
2. Marque **"Stream Logs"** e **"Include Timestamps"**
3. Monitore os logs durante o deploy e execução

## 🚨 Problemas Comuns

### Erro 400 (Bad Request)

- Verifique se `ALLOWED_HOSTS` inclui seu domínio Coolify
- Confirme se `CSRF_TRUSTED_ORIGINS` está correto
- Use `DEBUG=True` temporariamente para ver detalhes

### Erro "ModuleNotFoundError: No module named 'djproject'"

**Causa**: Você copiou o projeto mas não atualizou as referências internas.

**Solução**: Altere **TODOS** os arquivos listados na seção [📋 Copiando o Projeto](#-copiando-o-projeto-mudando-nome):
- `settings.py` → `ROOT_URLCONF` e `WSGI_APPLICATION`
- `wsgi.py` → `DJANGO_SETTINGS_MODULE`
- `asgi.py` → `DJANGO_SETTINGS_MODULE`
- `manage.py` → `DJANGO_SETTINGS_MODULE`
- `entrypoint.sh` → comando `gunicorn`

### Erro "exec format error"

- Problema com quebras de linha do `entrypoint.sh`
- O Dockerfile já inclui correção automática: `sed -i 's/\r$//'`

### Problemas de permissão de porta

- Use uma porta diferente: `python manage.py runserver 8001`
- Verifique se a porta 8000 não está em uso

## 📚 Referências

- [Tutorial original](https://fmacedo.com/posts/coolify-your-django-project/) por Francisco Macedo
- [Documentação do Coolify](https://coolify.io/docs/)
- [Documentação do Django](https://docs.djangoproject.com/)
- [Repositório de exemplo](https://github.com/franciscobmacedo/django-coolify-tutorial)

## ⚠️ Limitações do Coolify

**Zero Downtime Deployment**: Coolify não oferece deploy sem downtime facilmente com Docker. Para aplicações críticas, considere:

- Usar Swarm mode
- Outras soluções de deploy
- [Issue no GitHub](https://github.com/coollabsio/coolify/discussions/3767#discussioncomment-12040527)

## 📄 Licença

Este projeto é baseado no tutorial de Francisco Macedo e está disponível para fins educacionais.

## 🤝 Contribuição

Sinta-se à vontade para abrir issues ou enviar pull requests para melhorar este projeto.

---

**Desenvolvido com 💚 usando Django e Coolify**