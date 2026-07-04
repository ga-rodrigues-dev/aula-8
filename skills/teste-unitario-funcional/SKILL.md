---
name: teste-unitario-funcional
description: >
  Gera testes unitários funcionais para código Java/Spring Boot aplicando
  Partição de Equivalência (PE) e Análise de Valor-Limite (AVL). Use quando
  o usuário pedir testes unitários baseados em especificação/comportamento,
  sem olhar a estrutura interna do código.
---

# Objetivo
Derivar casos de teste a partir da ESPECIFICAÇÃO do método/classe alvo,
cobrindo classes de equivalência válidas e inválidas e seus valores-limite,
com rastreabilidade explícita entre cada teste e o critério que o originou.

# Contexto
- Aplicação web Java 17+ com Spring Boot, build Maven ou Gradle.
- Testes com JUnit 5 (Jupiter) + AssertJ; Mockito apenas para isolar
  dependências externas (repositórios, clients, serviços colaboradores).
- O alvo típico é um método de service, controller ou classe de domínio
  com regras de negócio e validações.

# Fluxo de trabalho
1. Leia a especificação/Javadoc/assinatura do método alvo e liste TODAS as
   variáveis de entrada (parâmetros, campos do DTO, estado relevante).
2. Para cada variável, derive as classes de equivalência VÁLIDAS e
   INVÁLIDAS (domínio, formato, faixa, nulidade, tamanho, enum, etc.).
3. Para cada classe com faixa ordenada, derive os valores-limite:
   mínimo−1, mínimo, mínimo+1, nominal, máximo−1, máximo, máximo+1.
4. Monte a tabela de casos de teste ANTES de escrever código (ver saída).
5. Gere um método de teste por caso, priorizando @ParameterizedTest
   (@CsvSource/@MethodSource) quando vários casos compartilham a lógica.
6. Verifique: toda classe de equivalência tem ≥1 teste? Todo limite tem
   seus vizinhos testados? Casos inválidos verificam a exceção/erro correto?

# Regras
- NUNCA invente regra de negócio: se a especificação for ambígua, registre
  a suposição em comentário `// SUPOSIÇÃO:` no teste.
- Um comportamento por teste; asserts com mensagem clara.
- Nomenclatura: metodo_condicao_resultadoEsperado
  (ex.: `criarUsuario_emailSemArroba_lancaValidationException`).
- @DisplayName em português citando a classe/limite coberto
  (ex.: "CE-3: email inválido — sem @").
- Casos inválidos devem assertar o TIPO da exceção e a mensagem/campo.
- Não usar dados aleatórios; valores devem tornar o limite óbvio.
- Não subir o contexto Spring (@SpringBootTest) para teste unitário;
  usar instâncias diretas + mocks.

# Formato de saída (obrigatório)
1. **Tabela de derivação** (markdown): ID | Variável | Classe/Limite |
   Válida? | Valor de teste | Resultado esperado.
2. **Código completo** da classe de teste, compilável, com imports.
3. **Explicabilidade**: cada @Test/@ParameterizedTest referencia o ID da
   tabela (CE-1, VL-2, ...) no @DisplayName ou comentário.
4. **Resumo final**: nº de classes de equivalência, nº de limites,
   lacunas conhecidas e suposições feitas.