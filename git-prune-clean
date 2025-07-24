#!/bin/bash

# Verifica se está em um repositório Git
if ! git rev-parse --is-inside-work-tree > /dev/null 2>&1; then
  echo "❌ Você não está dentro de um repositório Git."
  exit 1
fi

# Verifica se a branch develop existe
if ! git show-ref --verify --quiet refs/heads/develop; then
  echo "❌ A branch 'develop' não foi encontrada localmente."
  echo "💡 Execute: git checkout develop && git pull origin develop"
  exit 1
fi

echo "🔄 Atualizando referências remotas..."
git fetch --prune

echo ""
echo "🔍 Analisando branches para limpeza..."
echo "======================================"

# Pega a branch atual
current_branch=$(git branch --show-current)

# Lista todas as branches locais (exceto develop, master, main e a atual)
local_branches=$(git branch | grep -v "^\*" | grep -v -E "(develop|master|main)$" | sed 's/^ *//')

# Arrays para categorizar branches
safe_to_delete=()
maybe_safe=()
keep_branches=()

# Função para verificar se branch foi mergeada (incluindo squash)
check_branch_merged() {
  local branch=$1
  
  # Método 1: Verifica se é ancestral (merge tradicional)
  if git merge-base --is-ancestor "$branch" develop 2>/dev/null; then
    return 0
  fi
  
  # Método 2: Para squash merges - verifica se não há commits únicos
  local unique_commits=$(git rev-list --count "$branch" --not develop 2>/dev/null)
  if [ "$unique_commits" = "0" ]; then
    return 0
  fi
  
  # Método 3: Verifica se o último commit da branch tem árvore igual a algum commit em develop
  local branch_tree=$(git rev-parse "$branch^{tree}" 2>/dev/null)
  if [ -n "$branch_tree" ]; then
    # Procura nos últimos 50 commits de develop
    local develop_commits=$(git rev-list develop -50 2>/dev/null)
    for commit in $develop_commits; do
      local commit_tree=$(git rev-parse "$commit^{tree}" 2>/dev/null)
      if [ "$branch_tree" = "$commit_tree" ]; then
        return 0
      fi
    done
  fi
  
  return 1
}

# Analisa cada branch
for branch in $local_branches; do
  # Pula a branch atual
  if [ "$branch" = "$current_branch" ]; then
    continue
  fi
  
  # Verifica se existe no remoto
  remote_exists=$(git show-ref --verify --quiet "refs/remotes/origin/$branch" && echo "yes" || echo "no")
  
  # Verifica se foi mergeada
  if check_branch_merged "$branch"; then
    if [ "$remote_exists" = "no" ]; then
      safe_to_delete+=("$branch (órfã + mergeada)")
      echo "✅ DELETAR: $branch (mergeada e não existe no remoto)"
    else
      safe_to_delete+=("$branch (mergeada)")
      echo "✅ DELETAR: $branch (mergeada, mas ainda existe no remoto)"
    fi
  else
    if [ "$remote_exists" = "no" ]; then
      maybe_safe+=("$branch")
      echo "⚠️  CUIDADO: $branch (não mergeada e não existe no remoto)"
    else
      keep_branches+=("$branch")
      echo "📍 MANTER: $branch (não mergeada, existe no remoto)"
    fi
  fi
done

echo ""
echo "📊 RESUMO DA ANÁLISE:"
echo "===================="

if [ ${#safe_to_delete[@]} -gt 0 ]; then
  echo ""
  echo "✅ BRANCHES SEGURAS PARA DELETAR (${#safe_to_delete[@]}):"
  for branch in "${safe_to_delete[@]}"; do
    echo "   • $branch"
  done
fi

if [ ${#maybe_safe[@]} -gt 0 ]; then
  echo ""
  echo "⚠️  BRANCHES ÓRFÃS NÃO MERGEADAS (${#maybe_safe[@]}):"
  echo "   (Podem conter trabalho não salvo!)"
  for branch in "${maybe_safe[@]}"; do
    echo "   • $branch"
  done
fi

if [ ${#keep_branches[@]} -gt 0 ]; then
  echo ""
  echo "📍 BRANCHES ATIVAS (${#keep_branches[@]}):"
  echo "   (Não serão tocadas)"
  for branch in "${keep_branches[@]}"; do
    echo "   • $branch"
  done
fi

# Ação para branches seguras
if [ ${#safe_to_delete[@]} -gt 0 ]; then
  echo ""
  echo "❓ Deletar as ${#safe_to_delete[@]} branches mergeadas? [y/N]: "
  read -r answer
  
  if [[ "$answer" =~ ^[Yy]$ ]]; then
    echo ""
    echo "🗑️  Deletando branches mergeadas..."
    
    for branch_info in "${safe_to_delete[@]}"; do
      # Extrai apenas o nome da branch (remove a descrição entre parênteses)
      branch=$(echo "$branch_info" | cut -d' ' -f1)
      
      if git branch -d "$branch" 2>/dev/null; then
        echo "✅ $branch"
      elif git branch -D "$branch" 2>/dev/null; then
        echo "✅ $branch (forçado)"
      else
        echo "❌ Falha: $branch"
      fi
    done
    echo ""
    echo "✅ Limpeza das branches mergeadas concluída!"
  else
    echo "❌ Nenhuma branch foi deletada."
  fi
fi

# Ação para branches órfãs não mergeadas
if [ ${#maybe_safe[@]} -gt 0 ]; then
  echo ""
  echo "❓ O que fazer com as ${#maybe_safe[@]} branches órfãs NÃO mergeadas?"
  echo "1) Mostrar commits de cada uma para decidir"
  echo "2) Deletar TODAS (⚠️  PERIGOSO!)"
  echo "3) Manter todas"
  echo -n "Escolha [1-3]: "
  read -r choice
  
  case $choice in
    1)
      echo ""
      for branch in "${maybe_safe[@]}"; do
        echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
        echo "🔍 BRANCH: $branch"
        echo "📝 Commits únicos (não estão em develop):"
        
        if git log --oneline "$branch" --not develop --max-count=10 2>/dev/null | head -5; then
          echo ""
        else
          echo "   (Erro ao mostrar commits)"
        fi
        
        echo -n "❓ Deletar '$branch'? [y/N]: "
        read -r branch_answer
        
        if [[ "$branch_answer" =~ ^[Yy]$ ]]; then
          if git branch -D "$branch" 2>/dev/null; then
            echo "🔥 Deletada: $branch"
          else
            echo "❌ Erro ao deletar: $branch"
          fi
        else
          echo "✅ Mantida: $branch"
        fi
        echo ""
      done
      ;;
    2)
      echo ""
      echo "🔥 Deletando TODAS as branches órfãs não mergeadas..."
      for branch in "${maybe_safe[@]}"; do
        if git branch -D "$branch" 2>/dev/null; then
          echo "🔥 $branch"
        else
          echo "❌ Falha: $branch"
        fi
      done
      ;;
    *)
      echo "✅ Branches órfãs preservadas."
      ;;
  esac
fi

echo ""
echo "🎯 Limpeza finalizada!"
