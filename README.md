# Get Version from .csproj

![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/felipementel/get-version-from-csproj/tag-manually.yml?branch=main)

Este **reusable workflow** lê um arquivo `.csproj` e extrai a versão do projeto, combinando os valores das tags `<VersionPrefix>`, `<Version>` e `<VersionSuffix>`. O resultado pode ser utilizado em pipelines de CI/CD para versionamento automatizado de builds e releases.

---

## 🚀 **Como Usar**

Este workflow pode ser chamado dentro de outro workflow usando **`workflow_call`**.

### 📌 **Entrada (Inputs)**

| Nome      | Tipo   | Obrigatório | Descrição |
|-----------|--------|------------|-----------|
| `csproj` | `string` | ✅ Sim | Caminho completo do arquivo `.csproj` que contém as informações de versão. |

### 👤 **Saída (Outputs)**

| Nome    | Descrição |
|---------|-----------|
| `version` | Retorna a versão extraída do `.csproj`. O formato final depende das tags presentes no arquivo. |

---

## 📌 **Exemplo de Uso no GitHub Actions**

Aqui está um exemplo de como utilizar este reusable workflow em um outro workflow do seu repositório:

```yaml
name: Example Usage

on:
  push:
    branches:
      - main

jobs:
  get-project-version:
    uses: felipementel/get-version-from-csproj/.github/workflows/get-version-from-csproj.yml@main
    with:
      csproj: "./src/MyProject/MyProject.csproj"
    outputs:
      version: ${{ steps.get-version.outputs.version }}

  build-and-deploy:
    needs: get-project-version
    runs-on: ubuntu-latest
    steps:
      - name: Exibir versão extraída
        run: echo "Versão extraída: ${{ needs.get-project-version.outputs.version }}"

      - name: Criar Tag no GitHub
        run: |
          git tag v${{ needs.get-project-version.outputs.version }}
          git push origin v${{ needs.get-project-version.outputs.version }}
```

---

## 📌 **Exemplos de Extração de Versão**


Dado um `.csproj` com os seguintes valores:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <VersionPrefix>pre</VersionPrefix>
    <Version>1.0.3</Version>
    <VersionSuffix>rc</VersionSuffix>
  </PropertyGroup>
</Project>
```

O workflow retornará:

```
pre-1.0.3-rc
```

Dado um `.csproj` com os seguintes valores:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <Version>1.0</Version>
    <VersionSuffix>beta</VersionSuffix>
  </PropertyGroup>
</Project>
```

O workflow retornará:

```
1.0-beta
```

Caso o `.csproj` contenha apenas a tag `<Version>`:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <Version>2.3.1</Version>
  </PropertyGroup>
</Project>
```

O workflow retornará:

```
2.3.1
```

---

## 🛠 **Como Funciona Internamente?**

O script utiliza **`sed`** para buscar as versões dentro do `.csproj`:

```bash
versionPrefix=$(sed -n 's/.*<VersionPrefix>\(.*\)<\/VersionPrefix>.*/\1/p' ${{ inputs.csproj }})
version=$(sed -n 's/.*<Version>\(.*\)<\/Version>.*/\1/p' ${{ inputs.csproj }})
versionSuffix=$(sed -n 's/.*<VersionSuffix>\(.*\)<\/VersionSuffix>.*/\1/p' ${{ inputs.csproj }})
```

A variável **`tag`** é montada a partir dos valores extraídos e salva como **output do workflow**.

---

## 🏗 **Contribuindo**

💡 Quer sugerir melhorias ou relatar problemas? Siga estes passos:

1. **Fork** este repositório.
2. Crie uma nova branch:
   ```bash
   git checkout -b minha-melhoria
   ```  
3. Faça suas modificações e envie um commit.
   ```bash
   git commit -m "Melhoria no script de extração"
   ```  
4. Envie um **Pull Request**.

---

## 📄 **Licença**

Este projeto está licenciado sob a **MIT License**.

