# Kontrak Backend VPS Qiblatin (172.16.9.13, service TERPISAH dari CMS)

Satu sumber data buat APK kiosk + web. Endpoint baca PUBLIK (TV polling),
endpoint tulis BUTUH token admin.

## Baca (TV poll tiap 10 menit, cache offline bila gagal)

- `GET /api/qiblatin/config` → `kegiatan.json` (versi server = master bila VPS hidup)
- `GET /api/qiblatin/sholat?lat=-2.976&lon=104.775&tgl=2026-10-24`
  → `{subuh,terbit,dzuhur,ashar,maghrib,isya}` (Kemenag, fallback hitung lokal di TV)

## Tulis (admin, header `Authorization: Bearer <QIBLATIN_TOKEN>`)

- `PUT /api/qiblatin/config` body = `kegiatan.json` (validasi version+1)
- `POST /api/qiblatin/kegiatan` tambah sekali jalan `{tgl,judul,ket,ust}`
- `DELETE /api/qiblatin/kegiatan/:id`
- `PUT /api/qiblatin/jumat` `{muazin,khotib}` + `PUT /api/qiblatin/durasi` `{gap,fardhu}`
- `POST /api/qiblatin/refresh` → naikkan `version` (TV skip cache, ambil dalam 10 mnt)

## Deploy (ikut aturan CMS: JANGAN edit langsung di server)

- Folder/DB/port/service BEDa dari CMS. Auto-deploy via GitHub push → GH Actions.
- Env: `QIBLATIN_TOKEN` (admin), `QIBLATIN_PORT` (mis. 3101), DB sqlite/postgres tabel `kv(version, payload)`.
- Health: `GET /api/qiblatin/health` → `{ok,version}`.

## Fallback TV (tanpa backend pun jalan)

1. VPS hidup → pakai config server. 2. Mati → `kegiatan.json` GitHub raw.
3. Offline total → default bawaan APK (= isi file ini sekarang).
Jadwal: server → hitung lokal (rumus Kemenag di APK, ±1 mnt).
