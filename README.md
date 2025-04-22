const { makeWASocket, useSingleFileAuthState, DisconnectReason } = require('@adiwajshing/baileys');
const { Boom } = require('@hapi/boom');
const fs = require('fs');
const P = require('pino');

const { state, saveState } = useSingleFileAuthState('./auth_info.json');

async function startBot() {
    const sock = makeWASocket({
        logger: P({ level: 'silent' }),
        printQRInTerminal: true,
        auth: state
    });

    sock.ev.on('messages.upsert', async ({ messages, type }) => {
        if (type !== 'notify') return;
        const msg = messages[0];
        if (!msg.message || msg.key.fromMe) return;

        const from = msg.key.remoteJid;
        const name = msg.pushName || 'کاربر';
        const text = msg.message.conversation || msg.message.extendedTextMessage?.text;

        console.log(`[پیام جدید] ${name}: ${text}`);

        if (text === '!سلام') {
            await sock.sendMessage(from, { text: `سلام ${name}! من رفیق وفادارتم.` });
        } else if (text === '!راهنما') {
            await sock.sendMessage(from, { text: 'دستورات:\n!سلام\n!وضعیت\n!ساعت\n!خاموش' });
        } else if (text === '!وضعیت') {
            await sock.sendMessage(from, { text: 'ربات فعال است و به وظیفه‌اش عمل می‌کند.' });
        } else if (text === '!ساعت') {
            const now = new Date().toLocaleTimeString();
            await sock.sendMessage(from, { text: `ساعت فعلی: ${now}` });
        } else if (text === '!خاموش') {
            await sock.sendMessage(from, { text: 'ربات خاموش می‌شود. مراقب خودت باش!' });
            process.exit();
        } else {
            await sock.sendMessage(from, { text: `دستور "${text}" شناخته نشد. برای راهنما بزن !راهنما` });
        }
    });

    sock.ev.on('creds.update', saveState);

    sock.ev.on('connection.update', (update) => {
        const { connection, lastDisconnect } = update;
        if (connection === 'close') {
            const reason = new Boom(lastDisconnect?.error)?.output.statusCode;
            if (reason === DisconnectReason.loggedOut) {
                console.log('کاربر خارج شد. دوباره اجرا کنید.');
            } else {
                startBot();
            }
        }
    });
}

startBot();


{
  "name": "rafighpro",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "@adiwajshing/baileys": "^4.4.0",
    "@hapi/boom": "^10.0.0",
    "pino": "^8.0.0"
  }
}
