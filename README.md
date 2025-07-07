<div align='center'>Baileys Latest modifikasi by Tio</div><div align="center">Panduan komprehensif untuk menggunakan pustaka @whiskeysockets/baileys untuk berinteraksi dengan API Web WhatsApp.</div>🚀 MemulaiPanduan ini menyediakan semua yang Anda butuhkan untuk memulai dan menjalankan API WhatsApp Baileys.Contoh PenggunaanUntuk memulai dengan cepat, Anda dapat merujuk ke file contoh:Kode Contoh: Example/example.tsUntuk menjalankan contoh dasar:Arahkan ke direktori proyek Anda:cd path/to/Baileys
Instal dependensi yang diperlukan:npm install
Jalankan skrip contoh:node example.js
🔧 InstalasiAnda dapat menginstal Baileys menggunakan npm atau yarn.Versi Stabil (Disarankan):npm install @whiskeysockets/baileys
Versi Edge (Fitur Terbaru):Catatan: Versi ini mungkin memiliki ketidakstabilan.yarn add @whiskeysockets/baileys@latest
Setelah instalasi, impor pustaka ke dalam proyek Anda:const { default: makeWASocket } = require("@whiskeysockets/baileys");
📖 Daftar IsiMenghubungkan ke WhatsAppHubungkan dengan Kode QRHubungkan dengan Kode PemasanganManajemen SesiMenyimpan & Memulihkan SesiMenangani EventMengirim PesanPesan Teks & UtilitasPesan MediaFitur LanjutanMemodifikasi PesanManajemen GrupKueri PenggunaPengaturan PrivasiFungsionalitas KustomLisensi📱 Menghubungkan ke WhatsAppHubungkan akun Anda menggunakan kode QR atau Kode Pemasangan.Hubungkan dengan Kode QRHasilkan kode QR di terminal Anda untuk menautkan perangkat Anda.💡 Tips: Anda dapat menyesuaikan identifikasi browser. Lihat opsi Browser yang tersedia.const { default: makeWASocket, Browsers } = require("@whiskeysockets/baileys");

const sock = makeWASocket({
    // Sesuaikan nama browser
    browser: Browsers.ubuntu('My App'),
    printQRInTerminal: true
});
Hubungkan dengan Kode PemasanganGunakan kode pemasangan jika Anda tidak dapat memindai kode QR.⚠️ Penting: Nomor telepon harus hanya berisi angka (misalnya, 6281234567890), tanpa karakter khusus seperti +, -, atau ().const { default: makeWASocket } = require("@whiskeysockets/baileys");

const sock = makeWASocket({
    printQRInTerminal: false // Harus false untuk kode pemasangan
});

// Minta kode pemasangan
if (!sock.authState.creds.registered) {
    const phoneNumber = 'NOMOR_TELEPON_ANDA';
    const pairingCode = await sock.requestPairingCode(phoneNumber);
    console.log(`Kode Pemasangan Anda: ${pairingCode}`);
}
💾 Manajemen SesiMenyimpan & Memulihkan SesiHindari memindai ulang kode QR dengan menyimpan status otentikasi Anda.⚠️ Penting: useMultiFileAuthState adalah utilitas untuk menyimpan status otentikasi. Untuk sistem produksi, pertimbangkan solusi yang lebih kuat seperti database SQL atau No-SQL.const { default: makeWASocket } = require("@whiskeysockets/baileys");
const { useMultiFileAuthState } = require("@whiskeysockets/baileys");

async function connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info_baileys');

    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true
    });

    // Simpan kredensial setiap kali diperbarui
    sock.ev.on('creds.update', saveCreds);

    // ... tangani event lainnya
}

connectToWhatsApp();
🎧 Menangani EventBaileys menggunakan EventEmitter untuk mengelola event seperti pesan baru, pembaruan koneksi, dll.📚 Referensi: Daftar lengkap event dapat ditemukan di dokumentasi BaileysEventMap.Contoh: Mendengarkan Pesan BaruBerikut cara mendengarkan pesan masuk dan membalas secara otomatis.const { DisconnectReason } = require("@whiskeysockets/baileys");

sock.ev.on('connection.update', (update) => {
    const { connection, lastDisconnect } = update;
    if (connection === 'close') {
        const shouldReconnect = (lastDisconnect.error)?.output?.statusCode !== DisconnectReason.loggedOut;
        console.log('Koneksi ditutup. Menghubungkan kembali:', shouldReconnect);
        if (shouldReconnect) {
            connectToWhatsApp();
        }
    } else if (connection === 'open') {
        console.log('Koneksi dibuka!');
    }
});

sock.ev.on('messages.upsert', async (event) => {
    console.log('Menerima pesan:', JSON.stringify(event.messages, null, 2));

    for (const message of event.messages) {
        if (!message.key.fromMe) {
            await sock.sendMessage(message.key.remoteJid, { text: 'Halo!' });
        }
    }
});
✉️ Mengirim PesanFungsi sendMessage adalah alat utama Anda untuk mengirim semua jenis konten.// sintaks umum
sock.sendMessage(jid, content, options);
Pesan Teks & UtilitasPesan Teks:await sock.sendMessage(jid, { text: 'Halo, Dunia!' });
Membalas Pesan:await sock.sendMessage(jid, { text: 'Ini adalah balasan.' }, { quoted: originalMessage });
Menyebut Pengguna:await sock.sendMessage(jid, {
    text: 'Hai @6281234567890!',
    mentions: ['6281234567890@s.whatsapp.net']
});
Meneruskan Pesan:await sock.sendMessage(jid, { forward: messageToForward });
Reaksi:await sock.sendMessage(jid, {
    react: {
        text: '👍',
        key: message.key
    }
});
Pesan MediaAnda dapat mengirim media dari URL, path file, atau buffer.💡 Tips: Menggunakan stream atau URL lebih hemat memori daripada memuat seluruh file ke dalam buffer.Gambar:await sock.sendMessage(jid, {
    image: { url: './path/to/image.jpeg' },
    caption: 'Ini gambar untukmu!'
});
Video:await sock.sendMessage(jid, {
    video: { url: './path/to/video.mp4' },
    caption: 'Lihat video ini!'
});
Audio:Untuk kompatibilitas terbaik, konversi audio ke codec opus dalam wadah ogg.await sock.sendMessage(jid, {
    audio: { url: './path/to/audio.ogg' },
    mimetype: 'audio/ogg'
});
Pesan Sekali Lihat:await sock.sendMessage(jid, {
    image: { url: './path/to/image.jpeg' },
    caption: 'Ini akan hilang setelah dilihat.',
    viewOnce: true
});
✨ Fitur LanjutanJelajahi kemampuan lebih lanjut dari pustaka Baileys.Memodifikasi PesanMengedit Pesan:const sentMessage = await sock.sendMessage(jid, { text: 'Teks awal' });
await sock.sendMessage(jid, {
    text: 'Ini adalah teks yang diedit.',
    edit: sentMessage.key
});
Menghapus Pesan:const messageToDelete = await sock.sendMessage(jid, { text: 'Pesan ini akan dihapus.' });
await sock.sendMessage(jid, { delete: messageToDelete.key });
Manajemen GrupBuat Grup:const group = await sock.groupCreate('Grup Keren Baru', ['1234@s.whatsapp.net']);
console.log('Grup dibuat:', group.id);
Tambah/Hapus Peserta:await sock.groupParticipantsUpdate(
    groupId,
    ['participant_jid@s.whatsapp.net'],
    'add' // atau 'remove', 'promote', 'demote'
);
Perbarui Subjek Grup:await sock.groupUpdateSubject(groupId, 'Nama Grup Baru');
Kueri PenggunaPeriksa apakah Pengguna Ada:const [result] = await sock.onWhatsApp('1234567890@s.whatsapp.net');
if (result.exists) {
    console.log(`${result.jid} ada di WhatsApp.`);
}
Ambil Gambar Profil:const profilePicUrl = await sock.profilePictureUrl(jid, 'image'); // 'image' untuk resolusi tinggi
console.log(profilePicUrl);
Pengaturan PrivasiBlokir/Buka Blokir Pengguna:await sock.updateBlockStatus(jid, 'block'); // atau 'unblock'
Perbarui Privasi Terakhir Dilihat:await sock.updateLastSeenPrivacy('contacts'); // 'all', 'contacts', 'contact_blacklist', 'none'
✍️ Menulis Fungsionalitas KustomAnda dapat memperluas Baileys dengan mendengarkan event WebSocket mentah.Aktifkan Logging Debug:const sock = makeWASocket({
    logger: P({ level: 'debug' })
});
Daftarkan Callback:Dengarkan tag websocket tertentu untuk mengimplementasikan logika kustom.// Dengarkan node apa pun dengan tag 'edge_routing'
sock.ws.on('CB:edge_routing', (node) => {
    console.log('Menerima node edge_routing:', node);
});
📜 LisensiRepositori ini dilisensikan di bawah Lisensi GPL-3.0, karena menggunakan libsignal-node. Lihat file LICENSE untuk detail lebih lanjut.