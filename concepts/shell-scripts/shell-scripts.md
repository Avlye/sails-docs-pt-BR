# Shell scripts

Sails vem empacotado com [Whelk](https://github.com/sailshq/whelk), que permite executar funções JavaScript como shell scripts. Isso pode ser útil para executar tarefas agendadas (cron, agendador Heroku), processos worker, e qualquer outra personalização, scripts únicos que precisam de acesso aos modelos de sua aplicação Sails, configuração, e auxiliares.


### Seu primeiro script

Para adicionar um novo script, apenas crie um arquivo no diretório `scripts/` de sua aplicação.

```bash
sails generate script hello
```

Então, para executar, use:

```bash
sails run hello
```

> Se você precisa executar um script sem acesso global à interface de linha de comando `sails` (em uma Procfile, por exemplo), use `node ./node_modules/sails/bin/sails run hello`.

### Example

Aqui está um exemplo mais complexo que você pode ver em um app no mundo real:

```js
// scripts/send-email-proof-reminders.js
module.exports = {

  description: 'Envie um lembrete a todos os usuários recentes que ainda não confirmaram seus endereços de e-mail.',

  inputs: {
    template: {
      description: 'O nome de outro modelo de email para usar como uma substituição opcional.',
      type: 'string',
      defaultsTo: 'reminder-to-confirm-email'
    }
  },

  fn: async function (inputs, exits) {

    await User.stream({
      emailStatus: 'pending',
      emailConfirmationReminderAlreadySent: false,
      createdAt: { '>': Date.now() - 1000*60*60*24*3 }
    })
    .eachRecord(async (user, proceed)=>{
      await sails.helpers.sendTemplateEmail.with({
        template: 'reminder-to-confirm-email',
        templateData: {
          user: user
        },
        to: user.emailAddress
      });
      return proceed();
    });//∞

    return exits.success();

  }
};
```

Então você pode executar:

```bash
sails run send-email-proof-reminders
```

Para obter informações mais detalhadas sobre o uso, consulte [`whelk` README](https://github.com/sailshq/whelk/blob/master/README.md).

<docmeta name="displayName" value="Shell scripts">
<docmeta name="nextUpLink" value="/documentation/concepts/models-and-orm">
<docmeta name="nextUpName" value="Models and ORM">
