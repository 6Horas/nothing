const express = require('express');
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');

const DB_PATH = path.join(__dirname, 'db.json');
const PUBLIC_DIR = path.join(__dirname, '..', 'public');
const PORT = process.env.PORT || 3000;
const SESSION_TTL_MS = 1000 * 60 * 60 * 24 * 7; // 7 dias

const app = express();
app.use(express.json());
app.use(express.static(PUBLIC_DIR));

// ---------- helpers de banco de dados (arquivo JSON) ----------
function readDB() {
  return JSON.parse(fs.readFileSync(DB_PATH, 'utf-8'));
}
function writeDB(db) {
  fs.writeFileSync(DB_PATH, JSON.stringify(db, null, 2), 'utf-8');
}

// ---------- helpers de senha (scrypt, sem dependências externas) ----------
function hashPassword(password) {
  const salt = crypto.randomBytes(16).toString('hex');
  const hash = crypto.scryptSync(password, salt, 64).toString('hex');
  return `${salt}:${hash}`;
}
function verifyPassword(password, stored) {
  if (!stored) return false;
  const [salt, hash] = stored.split(':');
  const check = crypto.scryptSync(password, salt, 64).toString('hex');
  return crypto.timingSafeEqual(Buffer.from(hash, 'hex'), Buffer.from(check, 'hex'));
}

// ---------- middleware de autenticação de admin ----------
function requireAdmin(req, res, next) {
  const auth = req.headers.authorization || '';
  const token = auth.startsWith('Bearer ') ? auth.slice(7) : null;
  if (!token) return res.status(401).json({ error: 'Não autenticado.' });

  const db = readDB();
  const session = (db.sessions || []).find(s => s.token === token);
  if (!session || Date.now() - session.created > SESSION_TTL_MS) {
    return res.status(401).json({ error: 'Sessão expirada, faça login novamente.' });
  }
  req.db = db;
  next();
}

// ============================================================
// AUTH
// ============================================================

// Primeiro acesso: se ainda não existe senha de admin, define uma.
app.post('/api/setup', (req, res) => {
  const db = readDB();
  if (db.adminPasswordHash) {
    return res.status(400).json({ error: 'Uma senha de admin já foi configurada.' });
  }
  const { senha } = req.body;
  if (!senha || senha.length < 6) {
    return res.status(400).json({ error: 'A senha precisa ter ao menos 6 caracteres.' });
  }
  db.adminPasswordHash = hashPassword(senha);
  writeDB(db);
  res.json({ ok: true });
});

app.get('/api/setup-status', (req, res) => {
  const db = readDB();
  res.json({ needsSetup: !db.adminPasswordHash });
});

app.post('/api/login', (req, res) => {
  const db = readDB();
  const { senha } = req.body;
  if (!db.adminPasswordHash) {
    return res.status(400).json({ error: 'Nenhuma senha configurada ainda.' });
  }
  if (!verifyPassword(senha || '', db.adminPasswordHash)) {
    return res.status(401).json({ error: 'Senha incorreta.' });
  }
  const token = crypto.randomBytes(24).toString('hex');
  db.sessions = (db.sessions || []).filter(s => Date.now() - s.created < SESSION_TTL_MS);
  db.sessions.push({ token, created: Date.now() });
  writeDB(db);
  res.json({ token });
});

app.post('/api/logout', requireAdmin, (req, res) => {
  const auth = req.headers.authorization || '';
  const token = auth.slice(7);
  const db = req.db;
  db.sessions = (db.sessions || []).filter(s => s.token !== token);
  writeDB(db);
  res.json({ ok: true });
});

app.post('/api/change-password', requireAdmin, (req, res) => {
  const { senhaAtual, novaSenha } = req.body;
  const db = req.db;
  if (!verifyPassword(senhaAtual || '', db.adminPasswordHash)) {
    return res.status(401).json({ error: 'Senha atual incorreta.' });
  }
  if (!novaSenha || novaSenha.length < 6) {
    return res.status(400).json({ error: 'A nova senha precisa ter ao menos 6 caracteres.' });
  }
  db.adminPasswordHash = hashPassword(novaSenha);
  writeDB(db);
  res.json({ ok: true });
});

// ============================================================
// CONFIG (textos da página inicial)
// ============================================================

app.get('/api/config', (req, res) => {
  const db = readDB();
  res.json(db.config);
});

app.put('/api/config', requireAdmin, (req, res) => {
  const db = req.db;
  db.config = { ...db.config, ...req.body };
  writeDB(db);
  res.json(db.config);
});

// ============================================================
// ITENS
// ============================================================

app.get('/api/items', (req, res) => {
  const db = readDB();
  res.json(db.items);
});

app.post('/api/items', requireAdmin, (req, res) => {
  const db = req.db;
  const { categoria, nome, descricao, precoEstimado, links, imagemUrl } = req.body;
  if (!categoria || !nome) {
    return res.status(400).json({ error: 'Categoria e nome são obrigatórios.' });
  }
  const item = {
    id: crypto.randomUUID(),
    categoria,
    nome,
    descricao: descricao || '',
    precoEstimado: precoEstimado || '',
    imagemUrl: imagemUrl || '',
    links: Array.isArray(links) ? links : [],
    reservadoPor: null,
    reservadoEm: null
  };
  db.items.push(item);
  writeDB(db);
  res.status(201).json(item);
});

app.put('/api/items/:id', requireAdmin, (req, res) => {
  const db = req.db;
  const item = db.items.find(i => i.id === req.params.id);
  if (!item) return res.status(404).json({ error: 'Item não encontrado.' });
  const { categoria, nome, descricao, precoEstimado, links, imagemUrl } = req.body;
  if (categoria !== undefined) item.categoria = categoria;
  if (nome !== undefined) item.nome = nome;
  if (descricao !== undefined) item.descricao = descricao;
  if (precoEstimado !== undefined) item.precoEstimado = precoEstimado;
  if (imagemUrl !== undefined) item.imagemUrl = imagemUrl;
  if (links !== undefined) item.links = Array.isArray(links) ? links : [];
  writeDB(db);
  res.json(item);
});

app.delete('/api/items/:id', requireAdmin, (req, res) => {
  const db = req.db;
  const before = db.items.length;
  db.items = db.items.filter(i => i.id !== req.params.id);
  if (db.items.length === before) return res.status(404).json({ error: 'Item não encontrado.' });
  writeDB(db);
  res.json({ ok: true });
});

// ============================================================
// RESERVAS
// ============================================================

app.post('/api/items/:id/reserve', (req, res) => {
  const db = readDB();
  const item = db.items.find(i => i.id === req.params.id);
  if (!item) return res.status(404).json({ error: 'Item não encontrado.' });
  if (item.reservadoPor) {
    return res.status(409).json({ error: 'Este item já foi reservado por outra pessoa.' });
  }
  const nome = (req.body.nome || '').trim();
  if (!nome) return res.status(400).json({ error: 'Informe seu nome para reservar.' });
  item.reservadoPor = nome;
  item.reservadoEm = new Date().toISOString();
  writeDB(db);
  res.json(item);
});

// Convidado cancela a própria reserva (precisa digitar o mesmo nome) OU admin cancela qualquer uma.
app.post('/api/items/:id/unreserve', (req, res) => {
  const db = readDB();
  const item = db.items.find(i => i.id === req.params.id);
  if (!item) return res.status(404).json({ error: 'Item não encontrado.' });

  const auth = req.headers.authorization || '';
  const token = auth.startsWith('Bearer ') ? auth.slice(7) : null;
  const isAdmin = token && (db.sessions || []).some(s => s.token === token && Date.now() - s.created < SESSION_TTL_MS);

  if (!isAdmin) {
    const nome = (req.body.nome || '').trim().toLowerCase();
    if (!item.reservadoPor || nome !== item.reservadoPor.trim().toLowerCase()) {
      return res.status(403).json({ error: 'Digite o mesmo nome usado na reserva para poder cancelá-la.' });
    }
  }
  item.reservadoPor = null;
  item.reservadoEm = null;
  writeDB(db);
  res.json(item);
});

app.listen(PORT, () => {
  console.log(`Servidor do Chá de Panela rodando em http://localhost:${PORT}`);
});
