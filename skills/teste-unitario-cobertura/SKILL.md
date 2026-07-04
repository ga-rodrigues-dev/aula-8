---
name: teste-unitario-cobertura
description: >
  Gera e evolui testes unitários para Java/Spring Boot guiando-se por
  cobertura estrutural (JaCoCo): comandos, ramos/decisões e caminhos
  relevantes. Use quando o objetivo for aumentar/completar a cobertura de
  código existente com testes significativos.
---

# Objetivo
Partir do CÓDIGO-FONTE e do relatório de cobertura para identificar
comandos e ramos não exercitados e gerar testes que os cubram — sem
"inflar" cobertura com testes sem valor.

# Contexto
- Aplicação Java 17+ / Spring Boot com JaCoCo configurado
  (`./gradlew test jacocoTestReport` ou `mvn verify`).
- Critérios estruturais de referência: Todos-Comandos (statement),
  Todos-Ramos (branch/decision) e, quando indicado, condições compostas.
- Ponto de partida: suíte de testes existente + relatório HTML/XML do JaCoCo.

# Fluxo de trabalho
1. Rode (ou peça) o relatório JaCoCo atual e registre a cobertura BASE
   por classe (instruções % e ramos %).
2. Liste as classes/métodos com ramos não cobertos, priorizando lógica de
   negócio (services, domínio) sobre getters/config/boilerplate.
3. Para cada ramo não coberto, analise o predicado e derive a entrada/
   estado mínimo que força cada resultado (true/false; cada case do switch;
   cada catch; loops com 0, 1 e N iterações).
4. Gere os testes cobrindo esses ramos, SEMPRE com asserts sobre o
   comportamento observável (retorno, estado, exceção, interação mockada).
5. Re-execute a cobertura e reporte o delta (antes → depois).
6. Marque como INFEASIBLE ramos comprovadamente inalcançáveis, com
   justificativa, em vez de forçar testes artificiais.

# Regras
- PROIBIDO teste sem assert ou com assert trivial só para "pintar" linha.
- Cada teste novo indica em comentário o alvo estrutural:
  `// Cobre: ClasseX.metodoY — ramo else da linha 42 (desconto nulo)`.
- Priorizar cobertura de RAMOS sobre cobertura de linhas.
- Exceções: cobrir caminhos de erro (throw/catch) com o cenário real que
  os dispara, não com reflexão/hacks.
- Não alterar o código de produção para facilitar cobertura; se o código
  for intestável, registrar como recomendação de refatoração.
- Manter o estilo da suíte existente (nomenclatura, fixtures, builders).

# Formato de saída (obrigatório)
1. **Tabela de cobertura**: Classe | Instruções % (antes→depois) |
   Ramos % (antes→depois) | Alvos cobertos.
2. **Código completo** dos testes novos/alterados.
3. **Explicabilidade**: mapeamento teste → elemento estrutural coberto.
4. **Pendências**: ramos não cobertos restantes, infeasibles justificados
   e sugestões de refatoração para testabilidade.