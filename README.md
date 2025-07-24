# Git Branch Cleanup Tool 🧹

Uma ferramenta para limpar automaticamente branches locais que já foram mergeadas no Git, incluindo suporte para **squash merges**.

## 📋 Índice

- [Funcionalidades](#-funcionalidades)
- [Instalação](#-instalação)
- [Como Usar](#-como-usar)
- [O que o Script Faz](#-o-que-o-script-faz)
- [Exemplos de Uso](#-exemplos-de-uso)
- [Solução de Problemas](#-solução-de-problemas)
- [Contribuindo](#-contribuindo)

## ✨ Funcionalidades

- 🔍 **Detecção Inteligente**: Identifica branches mergeadas, incluindo squash merges
- 🛡️ **Proteção Automática**: Nunca deleta branches principais (`main`, `master`, `develop`)
- 📊 **Análise Clara**: Categoriza branches em seguras, perigosas e ativas
- 🤝 **Modo Interativo**: Permite revisar cada branch antes da deleção
- 🔄 **Atualização Automática**: Sincroniza com o repositório remoto antes da análise

## 🚀 Instalação

### Pré-requisitos

- Ubuntu 18.04+ (ou qualquer distribuição Linux)
- Git instalado e configurado
- Acesso de escrita ao diretório `/usr/local/bin` (ou outro diretório no PATH)

### Passo 1: Download do Script

```bash
# Clone este repositório ou baixe apenas o script
curl -o git-prune-clean https://raw.githubusercontent.com/alexandremsouza1/git-prune-clean/main/git-prune-clean

# Ou se você já tem o arquivo localmente, apenas renomeie:
# mv script_original.sh git-prune-clean
```

### Passo 2: Mover para o Diretório de Executáveis

```bash
# Mova o script para /usr/local/bin (requer sudo)
sudo mv git-prune-clean /usr/local/bin/

# Ou para ~/.local/bin (sem sudo, apenas para seu usuário)
mkdir -p ~/.local/bin
mv git-prune-clean ~/.local/bin/
```

### Passo 3: Dar Permissão de Execução

```bash
# Se moveu para /usr/local/bin
sudo chmod +x /usr/local/bin/git-prune-clean

# Se moveu para ~/.local/bin
chmod +x ~/.local/bin/git-prune-clean
```

### Passo 4: Atualizar o PATH (se necessário)

Se você usou `~/.local/bin`, adicione ao seu PATH:

```bash
# Adicione esta linha ao final do ~/.bashrc ou ~/.zshrc
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc

# Recarregue o terminal
source ~/.bashrc
```

### Passo 5: Verificar Instalação

```bash
# Teste se o comando está funcionando
git-prune-clean --help

# Ou simplesmente
git-prune-clean
```

## 📖 Como Usar

### Uso Básico

```bash
# Navegue até um repositório Git
cd /caminho/para/seu/repositorio

# Execute o script
git-prune-clean
```

### Comandos Git Equivalentes

O script funciona como um comando Git personalizado. Você também pode chamá-lo como:

```bash
git prune-clean
```

## 🔍 O que o Script Faz

### 1. Verificações Iniciais
- ✅ Confirma que você está em um repositório Git
- ✅ Verifica se a branch `develop` existe
- ✅ Atualiza as referências remotas (`git fetch --prune`)

### 2. Análise de Branches
O script categoriza suas branches locais em:

- **✅ SEGURAS PARA DELETAR**: Branches que foram mergeadas com `develop`
- **⚠️ CUIDADO**: Branches órfãs (não existem no remoto) e não mergeadas
- **📍 MANTER**: Branches ativas (existem no remoto e não foram mergeadas)

### 3. Proteção de Branches Principais
Nunca analisa ou deleta:
- `main`
- `master` 
- `develop`
- Branch atual

### 4. Detecção de Squash Merges
Identifica branches mergeadas via squash usando múltiplos métodos:
- Verificação de ancestralidade
- Contagem de commits únicos
- Comparação de árvores de arquivos

## 💡 Exemplos de Uso

### Exemplo 1: Limpeza Simples

```bash
$ git-prune-clean

🔄 Atualizando referências remotas...
🔍 Analisando branches para limpeza...
======================================
✅ DELETAR: feature/login (mergeada e não existe no remoto)
✅ DELETAR: bugfix/header (mergeada, mas ainda existe no remoto)
📍 MANTER: feature/new-dashboard (não mergeada, existe no remoto)

📊 RESUMO DA ANÁLISE:
====================
✅ BRANCHES SEGURAS PARA DELETAR (2):
   • feature/login (órfã + mergeada)
   • bugfix/header (mergeada)

❓ Deletar as 2 branches mergeadas? [y/N]: y

🗑️  Deletando branches mergeadas...
✅ feature/login
✅ bugfix/header

✅ Limpeza das branches mergeadas concluída!
🎯 Limpeza finalizada!
```

### Exemplo 2: Lidando com Branches Órfãs

```bash
$ git-prune-clean

⚠️  BRANCHES ÓRFÃS NÃO MERGEADAS (1):
   (Podem conter trabalho não salvo!)
   • experimental/ai-feature

❓ O que fazer com as 1 branches órfãs NÃO mergeadas?
1) Mostrar commits de cada uma para decidir
2) Deletar TODAS (⚠️  PERIGOSO!)
3) Manter todas
Escolha [1-3]: 1

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 BRANCH: experimental/ai-feature
📝 Commits únicos (não estão em develop):
a1b2c3d Add AI integration
e4f5g6h Update API endpoints
h7i8j9k Fix authentication

❓ Deletar 'experimental/ai-feature'? [y/N]: n
✅ Mantida: experimental/ai-feature
```

## 🔧 Solução de Problemas

### Comando não encontrado

```bash
# Verifique se o arquivo está no PATH
which git-prune-clean

# Verifique as permissões
ls -la /usr/local/bin/git-prune-clean
```

### Branch develop não encontrada

```bash
# O script precisa da branch develop. Crie ou baixe ela:
git checkout -b develop origin/develop
# ou
git checkout develop && git pull origin develop
```

### Erro de permissão ao mover para /usr/local/bin

Use `~/.local/bin` em vez disso:

```bash
mkdir -p ~/.local/bin
mv git-prune-clean ~/.local/bin/
chmod +x ~/.local/bin/git-prune-clean
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Script não detecta branches mergeadas com squash

Isso pode acontecer se:
- O squash merge foi feito há muito tempo (script verifica últimos 50 commits)
- Os commits foram modificados após o merge

Você pode verificar manualmente:
```bash
git log --oneline sua-branch --not develop
```

Se não retornar nada, a branch foi mergeada.

## 🤝 Contribuindo

1. Faça um fork do projeto
2. Crie uma branch para sua feature (`git checkout -b feature/nova-funcionalidade`)
3. Commit suas mudanças (`git commit -am 'Adiciona nova funcionalidade'`)
4. Push para a branch (`git push origin feature/nova-funcionalidade`)
5. Abra um Pull Request

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

## ⚠️ Avisos Importantes

- **Sempre faça backup** de branches importantes antes de usar o script
- **Teste primeiro** em um repositório de teste
- **Revise cuidadosamente** as branches que serão deletadas
- O script **nunca deleta** branches principais (`main`, `master`, `develop`)

## 📞 Suporte

Se encontrar problemas ou tiver sugestões:

1. Verifique a seção [Solução de Problemas](#-solução-de-problemas)
2. Abra uma [Issue](https://github.com/seu-usuario/seu-repo/issues)
3. Entre em contato com a equipe de desenvolvimento

---

**Feito com ❤️ para facilitar a vida dos desenvolvedores!**
