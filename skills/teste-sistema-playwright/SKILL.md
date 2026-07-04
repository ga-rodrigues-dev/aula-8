---
name: teste-sistema-playwright
description: >
  Gera testes de sistema (E2E) com Playwright para uma aplicação web
  Java/Spring Boot, cobrindo fluxos de negócio ponta a ponta pela UI.
  Use quando o usuário pedir testes de sistema, E2E ou de aceitação.
---

# Objetivo
Traduzir cenários de uso ponta a ponta em testes Playwright estáveis,
legíveis e independentes, validando o comportamento do sistema pela
perspectiva do usuário.

# Contexto
- SUT: aplicação web Spring Boot (frontend servido pela app ou SPA
  consumindo a API), rodando em URL base configurável.
- Playwright em Java (JUnit 5) por padrão; adaptar para TypeScript
  (@playwright/test) se o repositório alvo já usar essa stack.
- Cenários derivados de casos de uso/requisitos: derive-os com partição de
  equivalência no nível de fluxo (fluxo principal, alternativos, erro).

# Fluxo de trabalho
1. Identifique os fluxos de negócio críticos (login, CRUD principal,
   permissões, validações de formulário) e escreva-os como cenários
   Dado/Quando/Então antes de codificar.
2. Centralize o lifecycle e a configuração: Playwright → Browser →
   BrowserContext → Page; base URL, headless, slowMo e atributo de testId
   em um único ponto (fixture/classe base).
3. Organize a suíte com Page Objects: locators e ações de cada página em
   uma classe (um page object por tela significativa); extraia component
   objects para UI repetida (header, navbar); as ASSERTIONS ficam nos
   testes, não nos objects.
4. Injete page objects via fixtures — `test.extend` em TypeScript,
   classe base ou JUnit 5 extension em Java — eliminando instanciações
   repetidas (`new X(page)`) em beforeEach/@BeforeEach.
5. Prepare dados de teste via API/seed (não pela UI) sempre que possível;
   centralize a criação em factories fora dos testes; cada teste cria o
   que consome.
6. Escreva os testes usando web-first assertions com retry
   (`assertThat(locator)...` / `await expect(locator)...`); nunca
   `assertEquals(locator.textContent(), ...)` nem sleeps fixos.
7. Rode e verifique estabilidade (sem dependência de ordem; re-executável).

# Regras
- Localizadores na ordem: getByRole > getByLabel/getByPlaceholder >
  getByTestId; CSS/XPath frágeis são proibidos.
- Locators só existem dentro de page/component objects: se um seletor
  mudar, apenas um arquivo muda; testes descrevem intenção de negócio
  (ex.: `loginPage.login(...)`), nunca estrutura da página.
- Testes recebem page objects prontos via fixture; sem setup
  repetido nos testes.
- Testes independentes: sem estado compartilhado entre testes; limpeza
  ou isolamento por dados únicos (ex.: sufixo timestamp).
- Sem `Thread.sleep`/`waitForTimeout`; esperar por condição via assertions.
- @DisplayName descreve o cenário de negócio em português.
- Cobrir para cada fluxo: caminho feliz + pelo menos 1 alternativo +
  1 de erro (mensagem de validação visível ao usuário).
- Um cenário por teste; máximo ~1 fluxo de negócio por arquivo/classe.

# Formato de saída (obrigatório)
1. **Lista de cenários** Dado/Quando/Então com ID (SYS-1, SYS-2, ...).
2. **Código completo**: Page Object Model (+ component objects quando
   houver repetição), fixtures injetando os objects, factories de dados.
3. **Explicabilidade**: cada teste referencia o ID do cenário e o requisito
   /regra de negócio que valida.
4. **Notas de execução**: pré-requisitos (URL, seed, credenciais) e
   limitações conhecidas.