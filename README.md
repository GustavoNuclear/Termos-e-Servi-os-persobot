Termos de Serviço - MeuBot Discord

1. Uso responsável: Você concorda em usar o bot de forma responsável, sem abusos ou spam.
2. Dados: O bot pode coletar informações básicas para funcionar, mas não compartilha com terceiros.
3. Direitos: O bot é gratuito e não se responsabiliza por qualquer problema decorrente do uso.
4. Restrições: É proibido usar o bot para atividades ilegais.
5. Atualizações: Os termos podem ser atualizados sem aviso prévio.
6. Aceitação: Ao usar o bot, você concorda com esses termos.

 client.on('messageCreate', async message => {
  if(message.content === '!termos') {
    const termos = `Termos de Serviço:\n
    1. Uso responsável: ...
    2. Dados: ...
    3. Direitos: ...
    ...
    `;
    message.channel.send(termos);
  }
});

const termosMsg = await message.channel.send('Leia os termos e reaja com ✅ para aceitar.');

await termosMsg.react('✅');

const filter = (reaction, user) => reaction.emoji.name === '✅' && !user.bot;

const collector = termosMsg.createReactionCollector({ filter, max: 1, time: 60000 });

collector.on('collect', (reaction, user) => {
  message.channel.send(`${user} aceitou os termos!`);
  // Salvar que o user aceitou, liberar funções, etc
});

if(!userAceitou(user.id)) {
  return message.reply('Você precisa aceitar os termos antes de usar este comando! Use !termos para ver.');
}

const fs = require('fs');

let aceitos = { users: [] };

// Função para carregar os dados salvos
function carregarAceitos() {
  if(fs.existsSync('./aceitos.json')) {
    aceitos = JSON.parse(fs.readFileSync('./aceitos.json'));
  }
}

// Função para salvar dados no arquivo
function salvarAceitos() {
  fs.writeFileSync('./aceitos.json', JSON.stringify(aceitos, null, 2));
}

// Função para verificar se usuário aceitou
function userAceitou(id) {
  return aceitos.users.includes(id);
}

// Carrega os dados ao iniciar o bot
carregarAceitos();

// Evento mensagem
client.on('messageCreate', async message => {
  if(message.content === '!termos') {
    const termos = `Termos de Serviço:
1. Uso responsável...
2. Dados...
3. Direitos...
...`;
    // Envia mensagem e espera reação
    const termosMsg = await message.channel.send(termos + '\n\nReaja com ✅ para aceitar.');

    await termosMsg.react('✅');

    const filter = (reaction, user) => reaction.emoji.name === '✅' && user.id === message.author.id;

    const collector = termosMsg.createReactionCollector({ filter, max: 1, time: 60000 });

    collector.on('collect', reaction => {
      if(!userAceitou(message.author.id)) {
        aceitos.users.push(message.author.id);
        salvarAceitos();
        message.channel.send(`${message.author} aceitou os termos!`);
      }
    });
  }

  // Exemplo: comando protegido
  if(message.content === '!comando') {
    if(!userAceitou(message.author.id)) {
      return message.reply('Você precisa aceitar os termos antes de usar este comando! Use !termos para ver.');
    }
    message.channel.send('Você usou o comando protegido!');
  }
});
