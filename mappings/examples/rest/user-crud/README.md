# Mapeamentos CRUD de Usuários

## Stubs para POST /api/users

### user-create.json

- **Status**: 201 Created
- **Condição**: Request contém `name`, `email` E `cep`
- **Matching**: Usa `matchesJsonPath: "$.cep"` para verificar se o campo CEP EXISTE

### user-create-missing-cep.json

- **Status**: 400 Bad Request
- **Condição**: Request contém `name`, `email` mas NÃO contém `cep`
- **Matching**: Usa `matchesJsonPath: "$[?(!@.cep)]"` para verificar se o campo CEP NÃO EXISTE
- **Prioridade**: 1 (avaliado primeiro)

## Explicação da Expressão JSONPath

`"matchesJsonPath": "$[?(!@.cep)]"`

Isto **NÃO é uma expressão regular**. É uma **expressão de filtro JSONPath**:

- `$` = Raiz do objeto JSON
- `[?(...)]` = Cláusula de filtro (teste condicional)
- `!` = Operador lógico NOT (negação)
- `@.cep` = Propriedade `cep` do objeto atual
- `!@.cep` = "a propriedade `cep` NÃO existe"

**Resultado**: Faz match apenas se o campo `cep` está ausente do body da requisição

### Comparação

| Expressão | Significado | Faz Match |
|-----------|-------------|-----------|
| `$.cep` | "campo cep existe" | `{"cep": "123"}` ✅, `{}` ❌ |
| `$[?(!@.cep)]` | "campo cep NÃO existe" | `{}` ✅, `{"cep": "123"}` ❌ |
