# Base Documentation - Billing Plugin Ecosystem FeatherPanel

Dokumen ini merupakan panduan struktural dasar (base) untuk sistem billing plugin di FeatherPanel. Semua plugin berikut tersimpan di repositori ini:

- **billingcore** - Core billing management
- **billingplans** - Billing plans & subscriptions
- **pixpayz** - QRIS top-up payment gateway

---

## 1. Struktur Direktori Umum

Setiap plugin memiliki struktur standar berikut:

```
plugin-id/
├── conf.yml              # Konfigurasi metadata plugin
├── PluginId.php          # Kelas utama plugin (implementasi AppPlugin)
├── README.md             # Dokumentasi plugin
├── Migrations/           # File migrasi database (SQL)
├── Chat/                 # Model data (Chat ORM)
├── Controllers/          # Controller (Admin, User, Webhook)
├── Helpers/              # Helper class
├── Routes/               # Route definition
├── Frontend/             # Komponen frontend (sidebar.json, widgets.json, dist/)
└── Public/               # Aset publik (opsional)
```

---

## 2. Daftar Plugin

| Plugin | Identifier | Deskripsi | Versi | Dependensi |
|--------|-----------|-----------|-------|------------|
| BillingCore | `billingcore` | Powerful billing management core via FeatherPanel | 1.0.5 | `php=8.5`, `php-ext=pdo` |
| BillingPlans | `billingplans` | Sell servers as renewable billing plans with credit-based billing | 1.0.5 | `php=8.5`, `php-ext=pdo`, `plugin=billingcore` |
| PixPayz | `pixpayz` | Top up kredit BillingCore via QRIS PixPayz | 1.0.0 | `php=8.5`, `php-ext=pdo`, `php-ext=curl` |

---

## 3. Urutan Instalasi

Plugin harus diinstal dalam urutan berikut karena ketergantungan antar-plugin:

1. **billingcore** - Harus diinstal terlebih dahulu (foundation billing)
2. **billingplans** - Bergantung pada `billingcore`
3. **pixpayz** - Bergantung pada `billingcore` (untuk top-up kredit)

---

## 4. Format conf.yml

Contoh minimal konfigurasi plugin:

```yaml
plugin:
  name: NamaPlugin
  identifier: pluginid
  description: 'Deskripsi plugin'
  flags:
    - hasEvents
  version: 1.0.0
  target: v2
  author:
    - Nama Author
  icon: 'https://contoh.com/icon.png'
  requiredConfigs: {}
  dependencies:
    - php=8.5
    - php-ext=pdo
config: {}
```

---

## 5. Kelas Utama Plugin (AppPlugin)

Setiap plugin harus memiliki kelas utama yang mengimplementasi `App\Plugins\AppPlugin`:

```php
namespace App\Addons\pluginid;

use App\Plugins\AppPlugin;

class PluginId implements AppPlugin
{
    public static function processEvents(\App\Plugins\PluginEvents $event): void
    {
        // Event listeners
    }

    public static function pluginInstall(): void
    {
        // Instalasi (tabel dibuat via Migrations/)
    }

    public static function pluginUninstall(): void
    {
        // Cleanup
    }

    // Opsional
    public static function pluginUpdate(?string $oldVersion, ?string $newVersion): void
    {
        // Update logic
    }
}
```

---

## 6. Migrasi Database

File migrasi berada di `Migrations/` dengan format nama:
`YYYY-MM-DD-HH.MM-nama_migrasi.sql`

Contoh migrasi billingcore:

```sql
-- 2025-12-15-01.00-core.sql
CREATE TABLE IF NOT EXISTS featherpanel_billingcore_credits (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_uuid CHAR(36) NOT NULL,
    amount DECIMAL(15,2) NOT NULL DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## 7. Frontend

### sidebar.json
Definisi navigasi panel (client, admin, server):

```json
{
  "client": {
    "/billing/info": {
      "name": "Billing Information",
      "icon": "💳",
      "lucideIcon": "credit-card",
      "component": "/billingcore/dist/billing-info.html",
      "category": "client",
      "group": "Billing"
    }
  },
  "admin": {
    "/billingcore": {
      "name": "Billing Core",
      "icon": "💳",
      "lucideIcon": "credit-card",
      "component": "/billingcore/dist/admin.html",
      "category": "admin",
      "group": "Billing"
    }
  },
  "server": {}
}
```

### widgets.json
Definisi widget dashboard:

```json
[
	{
		"id": "billingcore-admin-user-billing-overview",
		"component": "billingcore/dist/admin-user-billing-widget.html",
		"enabled": true,
		"priority": 80,
		"page": "admin-users-edit",
		"location": "bottom-of-page",
		"pluginName": "Billing Core",
		"title": "Billing & resources",
		"icon": "BC"
	}
]
```

---

## 8. Alur Top-Up (PixPayz)

1. Pilih nominal top-up di menu **Billing → Top Up Saldo**
2. Tampilkan QRIS PixPayz
3. Sistem memeriksa status pembayaran setiap 4 detik
4. Saat status `paid`, kredit ditambahkan secara atomik di database
5. Invoice BillingCore ditandai `paid`

---

## 9. Keamanan

- API key disimpan di `PluginSettings`, tidak dikirim ke browser
- Webhook payload diverifikasi ulang ke API penyedia pembayaran
- Tidak ada kredensial sensitif yang disimpan di file plugin (kecuali di `.env` panel)

---

## 10. Instalasi Manual

```bash
# 1. Clone repositori
git clone https://github.com/zerotzyid/plygins.git
cd plygins

# 2. Salin plugin yang diinginkan ke direktori addons FeatherPanel
cp -r billingcore /var/www/html/storage/addons/
cp -r billingplans /var/www/html/storage/addons/
cp -r pixpayz /var/www/html/storage/addons/

# 3. Instal via panel admin → Plugins
#    Atau via CLI: php /var/www/html/cli module install billingcore
```

---

## 11. Lisensi

Semua plugin di lisensikan di bawah AGPL-3.0 (lihat LICENSE di masing-masing plugin).

---

Dokumen ini akan diperbarui seiring dengan perkembangan plugin billing ecosystem FeatherPanel.