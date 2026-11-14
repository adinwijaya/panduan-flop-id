# Panduan FLOP Network: Dari Nol Sampai Siap Testnet

### Catatan belajar — Bahasa Indonesia

> **Catatan jujur dari penulis:**
> Saya bukan programmer ahli. Panduan ini adalah catatan belajar saya — semua langkah di sini saya kerjakan dibantu AI agent (Muse) yang menjalankan perintahnya, saya yang mengarahkan. Semua dikerjakan pakai tools gratisan. Saya tulis apa adanya, termasuk gagalnya. Kalau ada yang salah atau bisa lebih baik, kabari saya.

> **⚠️ Peringatan paling penting:**
> JANGAN pernah membagikan mnemonic / seed phrase / private key kamu ke siapa pun — termasuk ke saya, ke AI manapun, atau ke orang yang mengaku "support". Simpan baik-baik di tempat aman. Yang boleh dishare hanya DID dan alamat publik (alamat wallet / SS58).

---

## Daftar Isi

1. [Pendahuluan](#1-pendahuluan)
2. [Devnet: Latihan](#2-devnet-latihan)
3. [Troubleshooting: Kendala di Devnet](#3-troubleshooting-kendala-di-devnet)
4. [Bikin DID: Identitas untuk Technocore & Testnet](#4-bikin-did-identitas-untuk-technocore--testnet)
5. [Siap Testnet: Checklist 24 November](#5-siap-testnet-checklist-24-november)

---

## 1. Pendahuluan

### FLOP Network itu apa?

FLOP Network adalah blockchain Layer-1 berbasis Substrate (Polkadot SDK) yang jadi **marketplace inference AI**. Sederhananya:

- **Miner** menyediakan GPU buat jalanin model AI.
- **Agen** (AI agent) membayar FLOP buat beli inference (minta model AI jawab pertanyaan / kerjain tugas).
- **Validator** memproduksi blok dan memverifikasi kerja miner.

Token FLOP **belum diluncurkan**. Pendirinya Arthur Hayes (co-founder BitMEX). Proyek ini masih tahap awal: devnet sudah jalan, testnet ditargetkan buka **24 November 2026** (tentative, bisa berubah — selalu cek pengumuman resmi `@flop_labs` di X dan `flop.finance`).

### Peta jalan (roadmap)

| Tahap | Status | Yang dihitung? |
|---|---|---|
| **Devnet** | Sudah jalan | ❌ TIDAK. Latihan doang. Tidak ada reward, tidak ada skor, tidak ada bobot airdrop. |
| **Testnet** | Target 24 Nov 2026, ~90 hari | ✅ YA. Ini satu-satunya jalur ke genesis airdrop. |
| **Mainnet + TGE** | Target Q1 2027 | — |

Sumber resmi: FAQ di `ask.flop.finance` — *"Devnet earns nothing: it is a throwaway network for proving setup, with no rewards and no airdrop weight."* Dan halaman testnet: testnet adalah *"the sole route to the genesis airdrop."*

### Dua jalur yang terpisah (jangan dicampur)

Ada **dua hal berbeda** yang sering bikin bingung. Pahami dulu sebelum mulai:

| | **Wallet Devnet** | **DID (Technocore)** |
|---|---|---|
| **Kunci** | sr25519 | Ed25519 |
| **Fungsi** | Latihan di devnet: enroll ke gateway, inference, baca receipt | Identitas kamu di Technocore & testnet |
| **Alamat** | SS58 (diawali `5...`) | `did:key:z6Mk...` + alamat SS58 Ed25519 |
| **Dibuat** | Bagian 2 (Devnet) | Bagian 4 (Bikin DID) |
| **Yang dihitung** | Tidak ada (latihan) | Bukan skor — tapi DID adalah syarat buat ikut testnet |

**Intinya:** wallet devnet buat latihan. DID buat identitas. Keduanya kunci yang berbeda, tidak bisa ditukar. Jangan coba "turunin" DID dari wallet devnet atau sebaliknya — tidak bisa, kurvanya beda.

### Yang dihitung vs yang tidak dihitung

**Yang dihitung (airdrop testnet):**
- Spend inference yang **settled** (receipt berstatus `settled`) di testnet. Agen dapat alokasi dari total 1,2 miliar FLOP (dari genesis 4,4 miliar) berdasarkan compute yang dibeli dalam sesi yang settle.

**Yang TIDAK dihitung:**
- Aktivitas devnet (berapa pun inference kamu di devnet, nol nilainya).
- Punya DID saja, atau posting di Technocore. Whitepaper §10: *"Activity on Technocore does not in itself constitute airdrop-eligible work. Key creation and message posting earn no allocation."*
- Pegang token faucet (dapat token gratis dari faucet tidak menciptakan hak alokasi).

Jadi: **devnet = latihan, DID = tiket masuk, testnet = yang dihitung.**

### Yang kamu butuhkan

- Komputer / cloud environment dengan Node.js (v20+) dan Python 3.12+.
- Akun GitHub (opsional, buat nyimpen catatan).
- Kemampuan buka terminal dan jalankan perintah. Tidak perlu jago coding — saya juga tidak.
- **Waktu dan kesabaran.** Enroll gateway itu susah (ada cap), gateway sering overload. Bukan sekali coba langsung berhasil.

---

## 2. Devnet: Latihan

> **Ingat:** semua yang kamu lakukan di bagian ini TIDAK menghasilkan reward/score/airdrop. Tujuannya cuma satu: **latihan** — supaya pas testnet buka, kamu sudah hafal alurnya (enroll → inference → receipt) dan tidak belajar dari nol.

### 2.1 Endpoint yang perlu diketahui

| Layanan | Alamat |
|---|---|
| Gateway (API inference) | `https://api.flop.network` |
| Faucet (token devnet gratis) | `https://faucet.flop.network/fund` |
| Chain RPC (devnet) | `wss://rpc.devnet.flop.network` |
| Dokumentasi | `https://docs.flop.network` |
| Technocore (chat agen) | `https://technocore.chat` |

### 2.2 Bikin wallet devnet

Wallet devnet memakai kunci **sr25519** (standar Substrate). Kamu bisa membuatnya dengan library `@polkadot/keyring` (Node.js) atau `subkey` (Rust).

Contoh dengan Node.js (`@polkadot/keyring` + `@polkadot/util-crypto`):

```js
const { Keyring } = require('@polkadot/keyring');
const { mnemonicGenerate } = require('@polkadot/util-crypto');

const kr = new Keyring({ type: 'sr25519', ss58Format: 42 });
const mnemonic = mnemonicGenerate(12);          // 12 kata — SIMPAN, JANGAN DISHARE
const pair = kr.addFromMnemonic(mnemonic);
console.log('Alamat:', pair.address);           // diawali 5... (SS58 prefix 42)
```

**Simpan mnemonic + alamat di file yang aman (mode 0600). Jangan pernah ketik mnemonic di chat / forum / ke siapa pun.**

### 2.3 Minta token dari faucet

Faucet devnet memberi **25 FLOP per alamat per 24 jam**, maksimal **300 FLOP per hari per IP**.

```bash
curl -X POST https://faucet.flop.network/fund \
  -H 'Content-Type: application/json' \
  -d '{"address": "<ALAMAT_WALLET_KAMU>"}'
```

Cek saldo:

```bash
# via API gateway (butuh key) atau via chain RPC
# cara cepat: cek di https://faucet.flop.network atau query chain
```

### 2.4 Enroll ke gateway (dapat Bearer key)

Enrollment adalah proses mendaftarkan wallet kamu ke gateway supaya dapat **Bearer key** (kunci API) untuk inference. Key ini terikat ke satu model, punya kuota (maksimal 20.000), dan berlaku **24 jam**.

Alurnya:

1. **Minta challenge:**
   ```
   GET /v1/enroll/wallet/challenge?address=<ALAMAT_WALLET_KAMU>
   ```
2. **Sign challenge** dengan kunci sr25519 wallet kamu (pesan harus sama persis).
3. **Kirim enrollment:**
   ```
   POST /v1/enroll/wallet
   { "address": "...", "challenge": "...", "signature": "0x..." }
   ```
4. Kalau berhasil (HTTP 201), kamu dapat Bearer key + info model + kuota + masa berlaku.

**Model devnet yang tersedia (per pengecekan saya):** `qwen/qwen2.5-0.5b-instruct` (kuota 20.000). Key kamu terikat ke model ini — tidak bisa dipakai buat model lain.

**Catatan penting tentang enroll:**
- Ada **cap harian bergilir (rolling 24 jam)** untuk allowance BARU. Kalau kena `503 wallet_enrollment_cap_reached`, itu artinya cap penuh — bukan salah kamu. Coba lagi nanti (lihat Troubleshooting).
- Key **tidak bisa di-renew**. Kalau key expired (lewat 24 jam) atau habis, satu-satunya cara dapat allowance baru adalah wallet baru. Re-enroll wallet yang sama tidak memperbarui allowance yang sudah expired.
- Key disimpan di file (misal `~/.flop/devnet-gateway.json`, mode 0600). Jangan tampilkan key di chat / log publik.

### 2.5 Jalankan inference

Dengan Bearer key, kamu bisa minta inference (chat completion, format OpenAI-compatible):

```bash
curl -X POST https://api.flop.network/v1/chat/completions \
  -H "Authorization: Bearer <KEY_KAMU>" \
  -H 'Content-Type: application/json' \
  -H 'Idempotency-Key: <ID_UNIK>' \
  -d '{
    "model": "qwen/qwen2.5-0.5b-instruct",
    "messages": [{"role": "user", "content": "Halo, siapa kamu?"}],
    "max_tokens": 128
  }'
```

**Tips dari pengalaman saya:**
- Pakai `max_tokens` kecil (128) — lebih cepat, lebih kecil kemungkinan timeout.
- Pakai `stream: true` kalau bisa — completion non-streaming sering mati kena timeout Cloudflare ~100 detik (lihat Troubleshooting).
- Selalu kirim `Idempotency-Key` yang unik tiap request.

### 2.6 Baca receipt & cek kuota

Setelah inference, **poll receipt** sampai statusnya jelas:

```
GET /v1/keys/me/receipts/<RECEIPT_ID>
```

Status receipt yang mungkin:
- `estimated` — masih diproses / menunggu settle.
- `settled` — **BERHASIL.** Spend tercatat. Ini satu-satunya status yang dihitung.
- `released` — gagal / dibatalkan. Spend yang sempat di-reserve dikembalikan. Tidak dihitung.

Cek kuota & pemakaian:

```
GET /v1/keys/me
```

Responsnya berisi `remaining_quota`, `spend`, dan masa berlaku key.

**Satu inference yang settle itu kecil** (contoh saya: 0,21 FLOP dari kuota 20.000). Jangan kaget — yang penting kamu paham alurnya. Di testnet nanti, yang dihitung adalah akumulasi spend yang settle selama ~90 hari.

---

## 3. Troubleshooting: Kendala di Devnet

> Bagian ini saya tulis dari pengalaman nyata saya — semua error di bawah ini benar-benar saya alami. Kalau kamu kena salah satunya, kamu tidak sendiri.

### 3.1 `503 wallet_enrollment_cap_reached` (enroll gagal)

**Artinya:** cap harian untuk allowance BARU sudah penuh. Ini **rolling 24 jam**, bukan reset tengah malam UTC — jadi tidak ada jam pasti kapan slot terbuka.

**Yang saya lakukan:**
- Coba lagi tiap 1 jam. Jangan spam berkali-kali dalam satu menit — itu tidak membantu dan bisa kena rate limit.
- Kalau gagal, tunggu jeda yang moderat dan bervariasi (misal 15–35 menit) baru coba lagi.
- Bersabar. Saya sendiri butuh beberapa kali percobaan selama berjam-jam sebelum akhirnya dapat slot.

**Yang TIDAK boleh dilakukan:** jangan buat banyak wallet baru cuma buat mengakali cap — faucet dan gateway melacak per-IP, dan model FLOP adalah "satu orang = satu skor". Wallet baru tanpa alasan yang jelas berisiko ditandai Sybil.

### 3.2 `429 BrokerOverloadedError: gateway channel pool is full` (inference gagal)

**Artinya:** gateway FLOP sedang overload — channel pool internal mereka penuh. Bukan salah key, script, atau koneksi kamu. Ini masalah di sisi server FLOP.

**Yang saya lakukan:**
- Retry dengan jeda bervariasi (15–35 menit). Jangan hantam gateway dengan retry rapat — itu tidak mempercepat dan terlihat seperti bot.
- Coba di jam yang berbeda. Gateway paling penuh di jam-jam sibuk.
- Terima bahwa ini di luar kendali kita. Kalau gateway penuh seharian, ya penuh seharian. Laporkan apa adanya, jangan pura-pura berhasil.

### 3.3 `524` / timeout Cloudflare (~100 detik)

**Artinya:** request completion non-streaming kamu mati di tengah jalan karena Cloudflare menutup koneksi setelah ~100 detik, padahal server mungkin masih memproses.

**Solusi yang saya temukan:**
- Pakai **streaming** (`stream: true`) — response mengalir sedikit demi sedikit, tidak menunggu sampai selesai baru dikirim, jadi tidak kena timeout 100 detik.
- Pakai `max_tokens` kecil (128) — response lebih pendek, lebih cepat selesai.
- Kalau tetap kena 524, cek receipt-nya: kadang request yang kena 524 ternyata sudah dijawab server dengan 429 lalu di-`released`. Jadi selalu cek receipt, jangan asumsi dari error HTTP saja.

### 3.4 Receipt `estimated` lama / tidak settle

**Artinya:** inference kamu sudah dijawab (HTTP 200), receipt dibuat dengan status `estimated`, tapi tidak kunjung jadi `settled`.

**Yang saya lakukan:**
- Poll receipt tiap 15 detik sampai 10 menit. Kalau setelah 10 menit masih `estimated`, kemungkinan besar tidak akan settle — catat dan lanjut ke percobaan berikutnya.
- Cek kuota: kalau `reserved_spend` masih tertahan di receipt `estimated`, kuota kamu berkurang sementara. Nanti kalau receipt jadi `released`, spend dikembalikan.

### 3.5 Receipt `released` (spend dikembalikan)

**Artinya:** inference gagal di tengah jalan (biasanya karena 429 atau 503 dari server). Spend yang sempat di-reserve (~1,6 FLOP) dikembalikan ke kuota kamu. **Tidak ada yang hilang permanen**, tapi juga tidak ada yang dihitung — receipt `released` = nol.

### 3.6 Script mati diam-diam (tidak ada log, tidak ada file hasil)

**Artinya:** script inference kamu berhenti tanpa pesan error — log kosong, file hasil tidak tertulis.

**Yang saya lakukan:**
- Cek apakah proses masih jalan (`ps aux | grep <nama-script>`).
- Cek apakah environment kamu restart / tidur. Kalau pakai cloud box gratisan, box bisa hibernate saat idle — cek responsif tidaknya sebelum attempt.
- Tambahkan logging yang jelas di script: tulis ke log setiap langkah (health check, request dikirim, response diterima, receipt dipoll). Kalau script mati, log terakhir memberi tahu di mana berhentinya.
- Jangan asumsi "tidak ada kabar = berhasil". Selalu verifikasi hasil (file hasil ada? receipt status apa?).

### 3.7 Key expired (lewat 24 jam)

**Artinya:** Bearer key kamu sudah lewat masa berlaku (24 jam sejak enroll). Inference akan gagal dengan `403` atau `devnet_enrollment_expired`.

**Solusi:** tidak ada jalan renew. Re-enroll wallet yang sama **tidak** memperbarui allowance yang sudah expired (ini terkonfirmasi dari spesifikasi resmi OpenAPI + GitHub issue #6: *"Re-enrolling rotates the bearer: spend and expiry are kept, and a revoked, expired, or exhausted allowance is never renewed."*). Satu-satunya cara dapat allowance baru adalah wallet baru — dan itu keputusan yang harus dipikirkan matang-matang (risiko Sybil, satu orang = satu skor).

### 3.8 Environment tidur / hibernate (cloud box gratisan)

**Artinya:** kalau kamu menjalankan script di cloud box gratisan, box bisa tidur (hibernate) saat idle. Kalau box tidur pas jam attempt, seluruh jadwal bisa gagal.

**Yang saya lakukan:**
- Selalu cek box responsif dengan satu perintah cepat sebelum attempt apa pun.
- Kalau tidak respons, coba lagi untuk membangunkannya. Kalau tetap tidak respons setelah beberapa kali coba, laporkan dan hentikan run itu — jangan diam.
- Jadwalkan "keep-alive" ringan kalau memungkinkan, atau terima bahwa hibernate adalah risiko dari tools gratisan.

### 3.9 Ringkasan error

| Error | Arti | Solusi |
|---|---|---|
| `503 wallet_enrollment_cap_reached` | Cap allowance baru penuh (rolling 24 jam) | Coba lagi tiap jam, jeda bervariasi |
| `429 BrokerOverloadedError` | Gateway overload | Retry jeda 15–35 menit, coba jam lain |
| `524` Cloudflare | Timeout ~100 detik (non-streaming) | Pakai streaming, `max_tokens` 128 |
| `403` | Wallet belum terdanai / key revoked / expired | Cek faucet, cek masa berlaku key |
| `402 devnet_allowance_exhausted` | Kuota habis | Tidak bisa di-renew; wallet baru (pikirkan risikonya) |
| `409` | Challenge sudah dipakai / expired | Minta challenge baru |
| `401` | Challenge / signature salah | Cek format signature, minta challenge baru |
| Receipt `released` | Inference gagal, spend dikembalikan | Tidak ada yang hilang; coba lagi |
| Receipt `estimated` lama | Belum settle | Poll sampai 10 menit, lalu catat dan lanjut |

---

## 4. Bikin DID: Identitas untuk Technocore & Testnet

### 4.1 DID itu apa?

DID (*Decentralized Identifier*) adalah identitas kamu di ekosistem FLOP. Bentuknya `did:key:z6Mk...` — diturunkan dari kunci publik **Ed25519**. Tidak ada server yang "menerbitkan" DID; kamu yang membuatnya sendiri dari keypair yang kamu pegang. Private key-nya hanya kamu yang punya, dan tidak bisa dicabut siapa pun.

**Fungsi DID:**
- Identitas kamu di **Technocore** (chat antar agen, `technocore.chat`).
- Syarat buat akses **faucet testnet** (hanya agen yang punya DID yang bisa klaim token testnet).
- Identitas yang dipakai di hari pertama testnet (dokumen resmi: *"an identity key (the same DID key used on TechnoCore)"*).

**Yang DID TIDAK lakukan:** DID saja tidak menghasilkan apa-apa. Tidak ada reward, tidak ada skor, tidak ada alokasi hanya karena punya DID. Whitepaper §10 jelas: pembuatan DID dan posting pesan tidak menghasilkan alokasi. DID adalah **tiket masuk**, bukan poin.

### 4.2 Generate DID

Kamu butuh:
- **Mnemonic 12 kata** (BIP39) → diturunkan jadi seed 32 byte.
- **Keypair Ed25519** dari seed itu.
- **did:key** = `"did:key:z"` + base58btc(`0xed01` + public key).
- **Alamat SS58** (prefix 42) dari public key yang sama — ini alamat yang muncul kalau kamu import mnemonic ke Polkadot.js / Talisman.

**Penting:** pakai library yang sama dengan yang dipakai Polkadot.js (`@polkadot/util-crypto` / `@polkadot/keyring`) supaya mnemonic yang kamu simpan, kalau di-import ke Polkadot.js/Talisman sebagai **Ed25519**, menghasilkan alamat yang SAMA persis. Saya pernah mengalami hasil yang tidak cocok karena memakai library yang berbeda untuk derivasi — jangan ulangi kesalahan itu. **Selalu verifikasi:** import mnemonic ke Polkadot.js (pilih crypto type Ed25519) dan pastikan alamat yang muncul sama dengan yang kamu catat.

Contoh dengan Node.js:

```js
const {
  mnemonicGenerate,
  mnemonicToMiniSecret,
  ed25519PairFromSeed,
  encodeAddress,
} = require('@polkadot/util-crypto');

// 1. Mnemonic baru (12 kata)
const mnemonic = mnemonicGenerate(12);

// 2. Seed 32 byte dari mnemonic (miniSecret = 32 byte pertama dari BIP39 seed)
const seed = mnemonicToMiniSecret(mnemonic);

// 3. Keypair Ed25519
const pair = ed25519PairFromSeed(seed);

// 4. did:key
const B58 = '123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz';
function base58btc(raw) {
  let n = BigInt('0x' + Buffer.from(raw).toString('hex'));
  let out = '';
  while (n > 0n) { out = B58[Number(n % 58n)] + out; n = n / 58n; }
  let leading = 0;
  for (const b of raw) { if (b === 0) leading++; else break; }
  return '1'.repeat(leading) + out;
}
const did = 'did:key:z' + base58btc(
  Buffer.concat([Buffer.from([0xed, 0x01]), Buffer.from(pair.publicKey)])
);

// 5. Alamat SS58 (prefix 42)
const ss58 = encodeAddress(pair.publicKey, 42);

console.log('DID :', did);
console.log('SS58:', ss58);
```

**Simpan hasilnya di file (mode 0600), misalnya `technocore-did.txt`:**
- DID (boleh dishare)
- Alamat SS58 (boleh dishare)
- Public key hex (boleh dishare)
- Mnemonic 12 kata (**JANGAN DISHARE**)
- Seed hex (**JANGAN DISHARE**)
- Cara import ke Polkadot.js / Talisman (pilih crypto type **Ed25519**)

**Verifikasi wajib sebelum lanjut:**
1. Import mnemonic ke Polkadot.js (pilih Ed25519) → alamat yang muncul harus SAMA dengan SS58 yang kamu catat.
2. Hitung did:key dari seed dengan tool independen (misal script referensi `agent-wallet.py` dari `flop-labs`) → DID yang muncul harus SAMA.

Kalau salah satu tidak cocok, **jangan lanjut** — ada yang salah di derivasi. Perbaiki dulu.

### 4.3 Publish DID note ke registry Technocore

Supaya DID kamu bisa ditemukan (di-resolve) agen lain dan faucet, publish **DID note** ke key-value store Technocore.

Caranya (konvensi sharded path, yang berlaku saat ini):
- Hitung fingerprint = 16 karakter hex pertama dari SHA-256 string DID lengkap.
- Shard = 2 karakter pertama, sisanya = 14 karakter berikutnya.
- Tulis note ke: `GET https://technocore.chat/kv/did-<shard>/<sisa>/set/<DID>?if_absent=1`

Contoh dengan Python:

```python
import hashlib, urllib.parse, urllib.request

did = 'did:key:z6Mk...'  # DID kamu
fp = hashlib.sha256(did.encode()).hexdigest()[:16]
shard, rest = fp[:2], fp[2:]
url = f'https://technocore.chat/kv/did-{shard}/{rest}/set/{urllib.parse.quote(did, safe="")}?if_absent=1'
with urllib.request.urlopen(url, timeout=30) as r:
    print(r.status, r.read().decode()[:200])
```

Verifikasi dengan membaca balik:

```bash
curl -s "https://technocore.chat/kv/did-<shard>/<sisa>"
# harus mengembalikan DID kamu
```

Catatan: path lama `/kv/did/<fingerprint>` punya batas jumlah note (sering penuh). Pakai **sharded path** (`/kv/did-<shard>/<sisa>`) yang tidak punya batas itu.

### 4.4 Signed check-in ke lobby

Posting pesan bertanda tangan ke lobby Technocore adalah bukti kamu memegang kunci DID itu. Pesan ditandatangani dengan skema: sign `room|nonce|text` dengan Ed25519, nonce = microsecond sejak epoch, signature dalam base64-urlsafe tanpa padding.

Contoh dengan script referensi `agent-wallet.py` (dari blog FLOP, `https://raw.githubusercontent.com/ktrxktr/agent-wallet/main/agent_wallet.py`):

```bash
python3 agent-wallet.py sign ./identity.key lobby "Hello, I am an autonomous agent. DID registered for FLOP testnet preparation."
```

Script itu mengirim POST ke `https://technocore.chat/r/lobby` dengan JSON `{did, sig, nonce, text, room}`. Server yang menerima akan mengembalikan pesan yang diposting.

**Catatan jujur:** lobby Technocore sangat ramai (ratusan pesan per menit, kebanyakan bot). Pesan check-in kamu akan cepat tenggelam. Tidak masalah — yang penting dan tahan lama adalah **DID note** di registry (Bagian 4.3), bukan pesan lobby.

### 4.5 Klaim faucet devnet via Technocore (opsional)

Sebagai langkah onboarding standar, kamu bisa klaim 10 dFLOP dari faucet devnet lewat Technocore:

```bash
python3 agent-wallet.py claim ./identity.key
```

Ini mengirim pesan bertanda tangan ke room `faucet` dengan teks:
```
dFLOP devnet faucet claim. DID: <DID_KAMU>. Requesting devnet tokens.
```

Pesan itu tercatat di room faucet dan DID kamu terpublikasikan di sana. **Catatan jujur dari pengalaman saya:** klaim ini tercatat, tapi drip 10 dFLOP-nya belum tentu langsung mendarat di alamat SS58 kamu (saldo saya tetap 0 setelah klaim). Ini token devnet — tidak dihitung buat apa-apa — jadi jangan khawatir kalau tidak langsung masuk.

### 4.6 Ringkasan: yang sudah kamu miliki setelah Bagian 4

- ✅ DID (`did:key:z6Mk...`) — identitas publik kamu.
- ✅ Alamat SS58 Ed25519 — akun yang muncul di Polkadot.js/Talisman.
- ✅ Mnemonic + seed tersimpan aman (file mode 0600, sudah di-backup).
- ✅ DID note terpublish di registry Technocore (bisa dibaca balik).
- ✅ Signed check-in lobby terkirim.
- ✅ Klaim faucet tercatat di room faucet.

**DID kamu sekarang siap dipakai di hari pertama testnet.**

---

## 5. Siap Testnet: Checklist 24 November

> Target testnet: **24 November 2026** (diumumkan Arthur Hayes 13 Nov 2026, disebut "tentative" oleh Flop Labs — bisa berubah). Selalu cek pengumuman resmi `@flop_labs` di X dan `flop.finance` sebelum berasumsi tanggalnya pasti.

### Checklist sebelum testnet buka

- [ ] **DID sudah siap** — mnemonic + seed tersimpan aman dan sudah di-backup (file mode 0600). Sudah terverifikasi: import ke Polkadot.js (Ed25519) menghasilkan alamat yang sama.
- [ ] **DID note terpublish** di registry Technocore (bisa dibaca balik).
- [ ] **Paham alur devnet** — sudah pernah enroll, inference, baca receipt, cek kuota di devnet (Bagian 2). Kalau belum, latihan dulu sebelum 24 Nov.
- [ ] **Tahu cara baca receipt** — hanya `settled` yang dihitung; `estimated` masih menunggu; `released` = gagal.
- [ ] **Tahu error yang umum** — 503 cap, 429 overload, 524 timeout (Bagian 3).
- [ ] **Tidak ada token FLOP palsu** — token FLOP belum diluncurkan. Apa pun yang mengaku token FLOP dan bisa dibeli sekarang adalah palsu. Jangan beli, jangan kirim uang ke siapa pun.
- [ ] **Seed/mnemonic tidak dishare** — tidak ke "support", tidak ke form, tidak ke siapa pun.

### Saat testnet buka (hari pertama)

1. **Klaim faucet testnet via Technocore** — hanya agen yang punya DID yang bisa mengaksesnya. Format klaim mengikuti pola yang sama seperti klaim devnet, tapi untuk testnet (cek pengumuman resmi untuk format pastinya saat testnet buka).
2. **Enroll wallet testnet ke gateway** — alur yang sama seperti devnet (challenge → sign → POST enroll). Key testnet terikat model, punya kuota, berlaku 24 jam.
3. **Inference sampai settle** — jalankan inference, poll receipt sampai `settled`. Hanya spend yang settle yang dihitung.
4. **Kerjakan tugas agen** — setelah enroll + inference berhasil, kerjakan tugas nyata (bukan sekadar 1x inference). Yang dihitung adalah akumulasi spend yang settle selama ~90 hari testnet.
5. **Pantau kuota & masa berlaku key** — key berlaku 24 jam; kalau expired, enroll ulang (aturan renewal mengikuti spesifikasi testnet yang berlaku saat itu).

### Yang dihitung di testnet (ringkasan)

- **Agen:** compute yang dibeli dalam sesi inference yang **settled**. Alokasi agen: 1,2 miliar FLOP (dari genesis 4,4 miliar).
- **Miner:** compute terverifikasi yang dilayani. Alokasi miner: 1,2 miliar FLOP.
- **Validator:** ranking testnet untuk kursi genesis. Alokasi validator: 1,2 miliar FLOP.
- **Ecosystem reserve:** 0,8 miliar FLOP — diskresioner (Flop Labs yang menilai, misal untuk kerja koordinasi/konektor; diumumkan sebelum snapshot).
- **Snapshot:** di akhir testnet, record dibekukan pada block height yang dipublish. Tidak ada rolling snapshot. Daftar alokasi + hash-nya dipublish sebelum genesis supaya peserta bisa cek dan ajukan sengketa.
- **Unlock (draft):** 1 FLOP terbuka per 3 FLOP dari principal yang di-spend dalam gross payable sesi inference yang settle (draft Yellow Paper, bisa berubah).

### Yang TIDAK dihitung (diulang supaya jelas)

- Aktivitas devnet. Pegang token faucet. Punya DID saja. Posting di Technocore. Wash activity antar wallet yang kamu kontrol.

---

## Penutup

Panduan ini saya tulis dari pengalaman nyata — termasuk semua gagalnya. Devnet itu latihan, DID itu tiket masuk, testnet itu yang dihitung. Kalau kamu mengikuti panduan ini sampai selesai, kamu sudah siap di hari pertama testnet — bukan karena kamu ahli, tapi karena kamu sudah latihan.

**Link resmi (selalu verifikasi dari sumber ini, bukan dari orang lain):**
- Website: `https://flop.finance`
- Dokumentasi: `https://docs.flop.network`
- Gateway API: `https://api.flop.network`
- Faucet: `https://faucet.flop.network`
- Technocore: `https://technocore.chat`
- X (Twitter): `@flop_labs`
- Whitepaper / Yellow Paper: `https://flop.finance/intro/yellowpaper/`

**Disclaimer:** Panduan ini adalah catatan belajar pribadi, bukan nasihat investasi dan bukan dokumen resmi Flop Labs. Aturan testnet masih draft dan bisa berubah — selalu cek pengumuman resmi. Token FLOP belum diluncurkan; jangan beli token apa pun yang mengaku FLOP. Saya tidak bertanggung jawab atas kehilangan dana / kunci / seed kamu — simpan baik-baik.

---

*Dibuat dari catatan belajar — dibantu AI agent (Muse), tools gratisan. Kalau ada yang salah atau bisa lebih baik, kabari saya.*
