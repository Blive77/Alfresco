# Integrar o Google Analytics 4 no Alfresco Share

Guia para adicionar rastreio (tracking) GA4 ao Alfresco Share, injetando o
`gtag.js` na página de login (para poderes testar logo) e no rodapé (que é
carregado em todas as páginas internas).

---

## Pré-requisitos

- Alfresco Share a funcionar (Community ou Enterprise). Confirma que o URL
  acaba em `/share`.
- Acesso ao servidor onde o Share corre (Tomcat).
- Uma conta Google Analytics.

---

## Passo 1 — Obter o Measurement ID (GA4)

1. Entra em <https://analytics.google.com> → **Admin** (ícone de engrenagem, canto inferior esquerdo).
2. Cria uma **Property** (se ainda não tiveres).
3. Cria um **Data Stream** do tipo **Web**. Aqui é importante meter o domínio
   corretamente, bem como se é `http://` ou `https://` (aponta ao URL do teu Alfresco).
4. Copia o snippet de instalação que inclui o **Measurement ID** — tem o formato `G-XXXXXXXX`.

Ao criar o Data Stream, a Google mostra logo as instruções de instalação e um
botão para **testar / verificar** o tag. Não precisas de usar o assistente para
adicionar o tag — o Passo 2 trata disso e deixa essa verificação pronta a passar.

---

## Passo 2 — Injetar na página de login e testar logo no Google

A página de login é o que carrega **sem** exigir autenticação, por isso é aqui
que metemos o snippet primeiro, para validar a property mal a crias.

### 2.1 — Snippet GA4

O ficheiro a editar é:

```
../tomcat/webapps/share/WEB-INF/classes/alfresco/site-webscripts/org/alfresco/components/guest/login.get.html.ftl
```

Adiciona ao final do ficheiro o bloco abaixo (substitui `G-XXXXXXXX` pelo teu ID):

```html
  <!-- Google tag (gtag.js) -->
  <script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXX"></script>
  <script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());
    gtag('config', 'G-XXXXXXXX');
  </script>
```

A seguir, faz restart ao serviço do Tomcat.

### 2.2 — Testar

1. Abre a página de login numa **janela anónima** (ou faz logout):
   `http://<host>:8080/share`. O `gtag.js` dispara sem precisares de credenciais.
2. Clica com o **botão direito → Ver código-fonte da página** e, com `Ctrl+F`,
   procura por `google` (ou `gtag`): deves encontrar o teu snippet do Google Analytics.
3. No assistente da property, o botão **testar / verificar** da Google funciona
   contra a página de login (que é pública), desde que o Alfresco esteja num
   URL acessível pela internet.

> **Sobre `localhost`:** o botão de verificação da Google tenta alcançar o teu
> URL **a partir dos servidores deles**, e `localhost` não é acessível pela
> internet — por isso essa verificação automática falha enquanto estiveres em
> local. Não é problema do tag: usa o **Realtime** + **DevTools** para validar.
> Quando publicares o Alfresco num domínio público, a verificação passa.

---

## Passo 3 — Rastrear o resto do Share (rodapé)

O rodapé (`footer`) carrega em todas as páginas internas, por isso uma única
inserção cobre o Share inteiro depois do login.

O ficheiro é:

```
../tomcat/webapps/share/WEB-INF/classes/alfresco/site-webscripts/org/alfresco/components/footer/footer.get.html.ftl
```

Cola o mesmo snippet no final do ficheiro e faz restart ao serviço ou à máquina
(nos meus testes, um reboot à máquina revelava-se mais rápido).

---

## Passo 4 — Verificar

1. Abre o Alfresco Share e navega por algumas páginas.
2. Na página do Google Analytics com a tua propriedade selecionada, vai ao
   separador **Realtime → Realtime overview**: vais ver o mapa com a tua
   localização e as métricas a serem registadas no Analytics.

---

## Notas

- **Verifica se a página tem o snippet:** se tiveres problemas de não chegar
  informação ao Google Analytics, inspeciona o código-fonte da página e confirma
  que o snippet do Google lá está — podes ter editado um ficheiro errado.
- Qualquer coisa, estou à disposição: fabio.a.felgueiras@gmail.com

---
