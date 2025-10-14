require('dotenv').config();
const { Client, GatewayIntentBits, Partials, PermissionsBitField } = require('discord.js');
const fs = require('fs');
const path = require('path');

const PREFIX = process.env.PREFIX || '.';
const OWNER_ID = process.env.OWNER_ID || '';
const TOKEN = process.env.DISCORD_TOKEN;
if (!TOKEN) {
  console.error('⚠  DISCORD_TOKEN missing in .env');
  process.exit(1);
}

const DATA_PATH = path.join(__dirname, 'data.json');
let db = {};

// load db
function loadDB() {
  if (!fs.existsSync(DATA_PATH)) {
    fs.writeFileSync(DATA_PATH, JSON.stringify({
      blacklist: [],
      whitelist: [],
      autoreplies: {},
      settings: { antispam:false, filterlinks:false, autorespond:false, floodprotect:false },
      logs: []
    }, null, 2));
  }
  db = JSON.parse(fs.readFileSync(DATA_PATH, 'utf8'));
}
function saveDB() {
  fs.writeFileSync(DATA_PATH, JSON.stringify(db, null, 2));
}
function logEvent(evt) {
  const entry = { ts: new Date().toISOString(), evt };
  db.logs.unshift(entry);
  if (db.logs.length > 200) db.logs.pop();
  saveDB();
}

// utils
function isLink(text) {
  return /(https?:\/\/|www\.)/i.test(text);
}

const client = new Client({
  intents: [ GatewayIntentBits.Guilds, GatewayIntentBits.GuildMessages, GatewayIntentBits.MessageContent ],
  partials: [ Partials.Channel ]
});

loadDB();

client.once('ready', () => {
  console.log(`✅ Nissi V2 DMF connecté en tant que ${client.user.tag}`);
});

// simple cooldown/flood protection per user (local, basique)
const lastMessage = new Map();

client.on('messageCreate', async message => {
  if (message.author.bot) return;

  // global filters
  const s = db.settings;

  // filter links
  if (s.filterlinks && isLink(message.content) && !db.whitelist.includes(message.author.id)) {
    await message.delete().catch(()=>{});
    logEvent(`Lien supprimé de ${message.author.tag} dans #${message.channel.name}`);
    return;
  }

  // flood protect simple
  if (s.floodprotect) {
    const key = `${message.guild?.id||'dm'}:${message.author.id}`;
    const now = Date.now();
    const prev = lastMessage.get(key) || 0;
    if (now - prev < 800) {
      await message.delete().catch(()=>{});
      logEvent(`Flood supprimé de ${message.author.tag}`);
      return;
    }
    lastMessage.set(key, now);
  }

  // autorespond
  if (s.autorespond) {
    for (const trigger in db.autoreplies) {
      if (message.content.toLowerCase().includes(trigger.toLowerCase())) {
        await message.channel.send(db.autoreplies[trigger]).catch(()=>{});
        logEvent(`Auto-reply "${trigger}" -> "${db.autoreplies[trigger]}" pour ${message.author.tag}`);
        break;
      }
    }
  }

  // command handling
  if (!message.content.startsWith(PREFIX)) return;
  const args = message.content.slice(PREFIX.length).trim().split(/\s+/);
  const cmd = args.shift().toLowerCase();
  const isOwner = message.author.id === OWNER_ID;

  switch(cmd) {
    case 'help':
    case 'menu': {
      const menu = `🛡 MARTIN FORDING DMF - Menu Commandes Sécurité (1 à 20) 🛡
.help - Voir ce menu
.status - Vérifier statut du bot
.antispam on/off - Activer/Désactiver l’anti-spam
.block [id] - Bloquer (simulé) un utilisateur
.unblock [id] - Débloquer (simulé)
.ban [id] - Signaler (simulé) un utilisateur
.purge [n] - Supprimer n messages (mod only)
.scan [msg] - Scanner message suspect
.filterlinks on/off - Filtrer liens dangereux
.log - Journal des menaces
.autorespond on/off - Réponses automatiques
.setreply [trigger] | [réponse] - Définir réponse auto
.delreply [trigger] - Supprimer réponse auto
.listreply - Lister réponses auto
.floodprotect on/off - Anti-flood
.whitelist add [id] - Ajouter numéro sûr (discord id)
.whitelist remove [id] - Retirer whitelist
.blacklist add [id] - Ajouter blacklist (simulé)
.blacklist remove [id] - Retirer blacklist
.shutdown - Éteindre le bot (owner only)
🔐 Nissi V2 DMF - Sécurité & Contrôle`;
      message.channel.send(menu);
      break;
    }

    case 'status': {
      const st = db.settings;
      message.channel.send(`Nissi V2 DMF en ligne.\nSettings: antispam=${st.antispam}, filterlinks=${st.filterlinks}, autorespond=${st.autorespond}, floodprotect=${st.floodprotect}`);
      break;
    }

    case 'antispam': {
      const param = args[0];
      if (!param || !['on','off'].includes(param)) return message.reply('Usage: .antispam on/off');
      db.settings.antispam = param === 'on';
      saveDB();
      message.reply(`antispam défini sur ${param}`);
      logEvent(`antispam ${param} par ${message.author.tag}`);
      break;
    }

    case 'filterlinks': {
      const p = args[0];
      if (!p || !['on','off'].includes(p)) return message.reply('Usage: .filterlinks on/off');
      db.settings.filterlinks = p === 'on';
      saveDB();
      message.reply(`filterlinks ${p}`);
      logEvent(`filterlinks ${p}`);
      break;
    }

    case 'autorespond': {
      const p = args[0];
      if (!p || !['on','off'].includes(p)) return message.reply('Usage: .autorespond on/off');
      db.settings.autorespond = p === 'on';
      saveDB();
      message.reply(`autorespond ${p}`);
      logEvent(`autorespond ${p}`);
      break;
    }

    case 'setreply': {
      const raw = message.content.slice((PREFIX + 'setreply').length).trim();
      const parts = raw.split('|').map(s=>s.trim());
      if (parts.length < 2) return message.reply('Usage: .setreply trigger | réponse');
      const trigger = parts[0].toLowerCase();
      const resp = parts.slice(1).join(' | ');
      db.autoreplies[trigger] = resp;
      saveDB();
      message.reply(`Réponse automatique ajoutée pour: "${trigger}"`);
      logEvent(`setreply "${trigger}" -> "${resp}"`);
      break;
    }

    case 'delreply': {
      const t = args.join(' ').toLowerCase();
      if (!t) return message.reply('Usage: .delreply trigger');
      if (db.autoreplies[t]) {
        delete db.autoreplies[t];
        saveDB();
        message.reply(`Réponse automatique supprimée: "${t}"`);
        logEvent(`delreply "${t}"`);
      } else message.reply('Trigger non trouvé');
      break;
    }

    case 'listreply': {
      const keys = Object.keys(db.autoreplies);
      if (!keys.length) return message.reply('Aucune réponse automatique définie.');
      const lines = keys.map(k => `• ${k} => ${db.autoreplies[k]}`).join('\n');
      message.channel.send(`Réponses automatiques:\n${lines}`);
      break;
    }

    case 'block': {
      const id = args[0];
      if (!id) return message.reply('Usage: .block [userId]');
      if (!db.blacklist.includes(id)) {
        db.blacklist.push(id);
        saveDB();
        message.reply(`ID ${id} ajouté à la blacklist (simulation).`);
        logEvent(`blocked (simulé) ${id} par ${message.author.tag}`);
      } else message.reply('Déjà en blacklist.');
      break;
    }

    case 'unblock': {
      const idu = args[0];
      if (!idu) return message.reply('Usage: .unblock [userId]');
      const idx = db.blacklist.indexOf(idu);
      if (idx !== -1) {
        db.blacklist.splice(idx,1);
        saveDB();
        message.reply(`ID ${idu} retiré de la blacklist.`);
        logEvent(`unblocked ${idu}`);
      } else message.reply('ID non trouvé en blacklist.');
      break;
    }

    case 'ban': {
      const idb = args[0];
      if (!idb) return message.reply('Usage: .ban [userId] (simulation de signalement)');
      logEvent(`reported (simulé) ${idb} par ${message.author.tag}`);
      message.reply(`Signalement simulé pour ${idb}. (Le bot n'exécute pas de signalement automatique réel.)`);
      break;
    }

    case 'purge': {
      if (!message.member.permissions.has(PermissionsBitField.Flags.ManageMessages)) {
        return message.reply('Permission requise: ManageMessages');
      }
      const n = parseInt(args[0]) || 10;
      const fetched = await message.channel.bulkDelete(n, true).catch(e=>{
        return null;
      });
      if (fetched) {
        message.channel.send(`Purge: ${fetched.size} messages supprimés.`).then(m=>setTimeout(()=>m.delete().catch(()=>{}),3000));
        logEvent(`purge ${fetched.size} messages par ${message.author.tag} dans #${message.channel.name}`);
      } else {
        message.reply('Impossible de purger (messages trop vieux ou erreur).');
      }
      break;
    }

    case 'scan': {
      const txt = args.join(' ');
      if (!txt) return message.reply('Usage: .scan [msg]');
      const suspicious = ['http','spam','senha','password','bank','phish','scam'];
      const found = suspicious.filter(w => txt.toLowerCase().includes(w));
      if (found.length) {
        message.reply(`ALERTE: mots suspects trouvés: ${found.join(', ')}`);
        logEvent(`scan ALERT: ${found.join(', ')} dans "${txt}"`);
      } else {
        message.reply('Message non-suspect détecté.');
        logEvent(`scan OK: "${txt}"`);
      }
      break;
    }

    case 'log': {
      const lines = db.logs.slice(0,20).map(l => `${l.ts} - ${l.evt}`);
      if (!lines.length) return message.reply('Aucun log.');
      message.channel.send(`Logs récents:\n${lines.join('\n')}`);
      break;
    }

    case 'floodprotect': {
      const p = args[0];
      if (!p || !['on','off'].includes(p)) return message.reply('Usage: .floodprotect on/off');
      db.settings.floodprotect = p === 'on';
      saveDB();
      message.reply(`floodprotect ${p}`);
      logEvent(`floodprotect ${p}`);
      break;
    }

    case 'whitelist': {
      const sub = args[0];
      const idw = args[1];
      if (!sub || !idw) return message.reply('Usage: .whitelist add/remove [id]');
      if (sub === 'add') {
        if (!db.whitelist.includes(idw)) { db.whitelist.push(idw); saveDB(); message.reply(`ID ${idw} ajouté en whitelist.`); logEvent(`whitelist add ${idw}`); }
        else message.reply('Déjà whitelisté.');
      } else if (sub === 'remove') {
        const j = db.whitelist.indexOf(idw);
        if (j!==-1) { db.whitelist.splice(j,1); saveDB(); message.reply(`ID ${idw} retiré de la whitelist.`); logEvent(`whitelist remove ${idw}`); }
        else message.reply('ID non trouvé en whitelist.');
      } else message.reply('Usage: .whitelist add/remove [id]');
      break;
    }

    case 'blacklist': {
      const sub2 = args[0];
      const idb2 = args[1];
      if (!sub2 || !idb2) return message.reply('Usage: .blacklist add/remove [id]');
      if (sub2 === 'add') {
        if (!db.blacklist.includes(idb2)) { db.blacklist.push(idb2); saveDB(); message.reply(`ID ${idb2} ajouté en blacklist (simulé).`); logEvent(`blacklist add ${idb2}`); }
        else message.reply('Déjà en blacklist.');
      } else if (sub2 === 'remove') {
        const j2 = db.blacklist.indexOf(idb2);
        if (j2!==-1) { db.blacklist.splice(j2,1); saveDB(); message.reply(`ID ${idb2} retiré de la blacklist.`); logEvent(`blacklist remove ${idb2}`); }
        else message.reply('ID non trouvé en blacklist.');
      } else message.reply('Usage: .blacklist add/remove [id]');
      break;
    }

    case 'shutdown': {
      if (!isOwner) return message.reply('Seul le propriétaire peut éteindre le bot.');
      message.channel.send('Arrêt du bot...').then(()=> {
        logEvent(`shutdown par ${message.author.tag}`);
        process.exit(0);
      });
      break;
    }

    default:
      message.reply('Commande inconnue. Tape .help pour le menu.');
  }

});

client.login(TOKEN);
