# TreinaWeb - Projeto prático de deploy Laravel VPS

## Passos para configurar a aplicação

### Clonar o repositório

```bash
git clone https://github.com/treinaweb/treinaweb-projeto-pratico-deploy-laravel-vps.git
```

### Acessar a pasta do projeto

```bash
cd treinaweb-projeto-pratico-deploy-laravel-vps
```

### Executar o composer install

```bash
composer install
```

### Criar o arquivo .env

```bash
cp .env.example .env
```

### Configurar o banco de dados no arquivo .env

```bash
DB_CONNECTION=mysql
DB_HOST=endereço_do_servidor
DB_DATABASE=nome_do_banco_de_dados
DB_USERNAME=usuário_do_banco_de_dados
DB_PASSWORD=senha_do_usuário
```

### Executar as migrations

```bash
php artisan migrate
```

### Gerar a chave da aplicação

```bash
php artisan key:generate
```

## Configurar o deploy

As instruções de deploy estão no arquivo [.github/Deploy.md](.github/Deploy.md).