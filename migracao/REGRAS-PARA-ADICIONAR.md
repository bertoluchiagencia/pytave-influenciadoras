# Regras para adicionar no Firebase (painel de revendedoras)

Estas regras **se somam** às que já existem para o site de influenciadoras — não
apague nada que já está lá, só adicione os trechos abaixo.

## 1. Firestore

1. Abra https://console.firebase.google.com/project/pytave-influenciadoras/firestore/rules
2. Dentro do bloco `match /databases/{database}/documents { ... }`, cole este
   trecho (em qualquer lugar dentro das chaves, antes do `}` final):

```
    // ---- Revendedoras ----
    match /revendedoras/{token} {
      allow read: if true;
      allow write: if request.auth != null
                   && request.auth.token.email is string
                   && request.auth.token.email.matches('.*@bertoluchiagencia[.]com[.]br$');

      match /notas/{notaId} {
        allow read: if true;
        allow write: if request.auth != null
                     && request.auth.token.email is string
                     && request.auth.token.email.matches('.*@bertoluchiagencia[.]com[.]br$');
      }
      match /vendas/{vendaId} {
        allow read: if true;
        allow create: if request.resource.data.peca is string
                       && request.resource.data.cliente is string
                       && request.resource.data.valor is number
                       && request.resource.data.valor > 0
                       && request.resource.data.valor < 100000;
        allow update, delete: if request.auth != null
                     && request.auth.token.email is string
                     && request.auth.token.email.matches('.*@bertoluchiagencia[.]com[.]br$');
      }
      match /pagamentos/{mes} {
        allow read: if true;
        allow write: if request.auth != null
                     && request.auth.token.email is string
                     && request.auth.token.email.matches('.*@bertoluchiagencia[.]com[.]br$');
      }
    }
    match /revConfig/{doc} {
      allow read: if true;
      allow write: if request.auth != null
                   && request.auth.token.email is string
                   && request.auth.token.email.matches('.*@bertoluchiagencia[.]com[.]br$');
    }
```

3. Clique em **Publicar**.

## 2. Storage (upload das notas de remessa)

1. Abra https://console.firebase.google.com/project/pytave-influenciadoras/storage/rules
2. Dentro do bloco `match /b/{bucket}/o { ... }`, cole:

```
    match /revendedoras/{token}/notas/{fileName} {
      allow read: if true;
      allow write: if request.auth != null
                   && request.auth.token.email is string
                   && request.auth.token.email.matches('.*@bertoluchiagencia[.]com[.]br$')
                   && request.resource.size < 20 * 1024 * 1024;
    }
```

3. Clique em **Publicar**.

Pronto — não precisa de terminal, CLI nem login por linha de comando. Tudo é
feito pelo navegador, com a mesma conta Google que já usa para administrar o
site das influenciadoras.

## O que essas regras garantem

- Qualquer pessoa com o **link/código** de uma revendedora específica consegue
  ver o estoque, as vendas e o relatório dela (mesmo nível de proteção que já
  existe hoje no site das influenciadoras — o "código de acesso" funciona como
  senha).
- Só a revendedora com aquele link consegue **registrar uma venda** (e só
  registrar — não editar nem apagar vendas já lançadas).
- Só a equipe Pytave (e-mail `@bertoluchiagencia.com.br`, logada com o Google)
  pode subir notas de remessa, editar comissão, marcar comissão como paga, ou
  apagar qualquer coisa.
