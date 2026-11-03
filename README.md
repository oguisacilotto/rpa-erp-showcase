![RPA de Fechamento](assets/banner.svg)

# RPA de Fechamento — Fullpack ERP

> Robô que **opera o ERP pela própria tela** para gerar, todo mês, o pacote de relatórios do **fechamento** — sem ninguém clicando. O ERP é um sistema **.NET + WinDev sem API**, então o robô trabalha pela interface, como uma pessoa faria: abre menus, digita, lê a grade e exporta.

![Python](https://img.shields.io/badge/Python-3-3776AB?logo=python&logoColor=white)
![Automação de tela](https://img.shields.io/badge/automa%C3%A7%C3%A3o-de%20tela%20(GUI)-D10524)
![Telegram](https://img.shields.io/badge/Telegram-notifica%C3%A7%C3%B5es-26A5E4?logo=telegram&logoColor=white)
![ERP read-only](https://img.shields.io/badge/valida%C3%A7%C3%A3o-somente%20leitura-1E7F4F)
![Status](https://img.shields.io/badge/status-em%20produ%C3%A7%C3%A3o-success)

> **Esta é uma vitrine.** O código de produção, as credenciais, as coordenadas de tela e qualquer relatório com dados financeiros ficam em repositório **privado**. Aqui mostramos só a ideia e as decisões de engenharia — nenhuma tela real do ERP, nenhum número.

---

## O problema

O fechamento mensal exigia uma pessoa sentada no ERP por horas: abrir dezenas de telas, aplicar sempre os mesmos filtros, exportar cada relatório, conferir, repetir. Trabalho repetitivo, demorado e fácil de errar quando o volume aperta.

O ERP não tem API nem banco aberto para escrita — é um aplicativo desktop **.NET sobre runtime WinDev**. Integrar "por baixo" não era opção. Então automatizamos **por cima**: o robô controla a tela.

---

## Como funciona

![Fluxo do robô](assets/fluxo.svg)

1. **Gatilho** — uma agenda mensal (ou um comando no Telegram) dispara a rodada.
2. **Operar a tela** — um conjunto de *receitas* controla o ERP: posiciona a janela, abre o menu certo, preenche filtros, espera o popup de carregamento, lê a grade e manda exportar.
3. **Montar relatórios** — *extratores* transformam o que saiu do ERP nos arquivos finais do fechamento (`.xlsx` / `.pdf`), já no formato que a área espera.
4. **Entregar e conferir** — o robô avisa pelo Telegram a cada etapa e, ao final, uma **validação somente‑leitura** (apenas `SELECT`) cruza os números para garantir que bate.

---

## Destaques de engenharia

**Automação de GUI determinística.** Cada tela do ERP vira uma *receita* versionada. Quando um update do ERP desloca um menu, conserta‑se a receita — não o robô inteiro. As receitas sabem esperar o sistema ("Carregando…") em vez de confiar em tempo fixo, o que elimina a maior fonte de erro em automação de tela.

**Robustez de verdade.** Cada passo valida que chegou onde deveria antes de seguir; se uma tela não aparece, o robô para e avisa em vez de "clicar no escuro". Detalhes chatos do mundo real — layout de teclado ABNT2, popups que roubam o foco, grades que paginam — são tratados explicitamente.

**Conferência somente‑leitura.** A checagem dos números usa uma conexão de banco **bloqueada para escrita**: qualquer `INSERT`/`UPDATE`/`DELETE` é barrado no próprio código. O robô nunca altera o sistema — só lê para comparar. Isso é regra de segurança do projeto, não um detalhe.

**Avisa o tempo todo.** Início, progresso, conclusão e falhas vão para o Telegram, com a saída organizada por rodada. Quem acompanha o fechamento sabe em tempo real onde está — sem abrir a VM.

**Roda sozinho.** Opera numa VM dedicada, dispara pela agenda e sobe o monitor automaticamente. Nas últimas rodadas, gerou e conferiu **todo o pacote de relatórios do fechamento** sem intervenção.

---

## Stack

| Camada | Tecnologia |
| --- | --- |
| Linguagem | Python 3 |
| Automação | Controle de GUI (tela/teclado/mouse), *receitas* por tela do ERP |
| Alvo | Fullpack ERP — .NET sobre runtime WinDev (sem API) |
| Relatórios | Extratores que geram `.xlsx` e `.pdf` do fechamento |
| Validação | Conexão de banco **somente‑leitura** (apenas `SELECT`) para conferir os números |
| Notificação | Telegram (início, progresso, conclusão, falhas) |
| Operação | VM dedicada, disparo por agenda, monitor automático |

---

## Estrutura (resumo)

```
rodar_fechamento.py   # orquestrador: conduz a rodada do começo ao fim
receitas_gui/         # uma receita por tela do ERP (abrir, filtrar, exportar)
relatorios/           # extratores que montam os arquivos finais + conferência read-only
```

> As coordenadas de tela, os nomes dos relatórios e a saída (`retorno/`, com dados do fechamento) **não** estão aqui — ficam no repositório privado e no `.gitignore`.

---

## Status

Em produção, roda no fechamento mensal. Código‑fonte, credenciais e relatórios são **privados** — este repositório existe apenas para mostrar como o robô foi construído.

<sub>Feito pela equipe de TI da Betel Sport.</sub>
