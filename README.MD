# LegalDoc RAG – Intelligent Contract Review & Q&A

## 1. Background / Context
Perusahaan kecil dan menengah sering menandatangani kontrak (sewa, vendor, karyawan) tanpa memahaminya sepenuhnya. Konsultan hukum mahal (Rp 1-5 juta per jam). Akibatnya, klausul merugikan terlewat.

## 2. Problem Statement
- Dokumen hukum panjang (20-100 halaman), tidak praktis dibaca manual.
- Istilah hukum rumit (legalese) membuat non-lawyer bingung.
- Risiko litigasi karena tidak menyadari klausul berbahaya (misal: indemnifikasi sepihak, auto-renewal).
- Tidak ada alat yang bisa menjawab pertanyaan spesifik tentang kontrak secara instan.

## 3. Goals / Objectives
- Menurunkan waktu review kontrak dari 3 hari menjadi 15 menit.
- Mendeteksi 10+ jenis klausul berisiko dengan akurasi >90%.
- Memberikan saran perbaikan dalam bahasa sederhana.
- Menyediakan Q&A interaktif (chat) tentang kontrak.

## 4. Proposed Solution
**RAG (Retrieval-Augmented Generation) khusus dokumen legal** dengan:
- OCR untuk scan kontrak.
- Chunking cerdas (berdasarkan pasal, bukan arbitrary).
- Fine-tuned embedding model (legal-domain).
- LLM untuk menjawab pertanyaan dan menyorot klausul.

## 5. System Design / Architecture
[User upload PDF/DOCX] → [Preprocessing (OCR jika perlu)]
→ [Chunking by clauses] → [Embedding (legal-BERT)] → [Vector DB (Qdrant)]
↓
[User query: "Apa risiko klausul indemnifikasi?"] → [Retrieve top 5 relevant chunks]
↓
[LLM (GPT-4 / Llama 3 70B)] → [Generate answer + highlight lokasi di dokumen]

**Tambahan:** Risk detector module (rule-based + classifier) untuk menandai klausul merah/kuning.

## 6. Detailed Design
**Chunking cerdas:**
- Parse dokumen dengan `docling` atau `pypdf` untuk identifikasi nomor pasal.
- Setiap chunk = satu pasal (atau sub-pasal). Metadata: nomor pasal, judul bab.

**Legal embedding:**
- Gunakan model `nlpaueb/legal-bert-base-uncased` (fine-tuned di dataset CUAD).
- Embedding dimension 768.

**Risk detection:**
- Daftar klausul berbahaya (indemnifikasi, perubahan sepihak, yurisdiksi asing, dll).
- Untuk setiap pasal, hitung cosine similarity dengan template klausul berbahaya.
- Jika similarity > 0.75 → tag "Red Flag".

**Q&A:**
- User bisa tanya: "Apakah saya bisa terminate kontrak lebih awal?"
- RAG retrieve pasal yang membahas termination.
- LLM menjawab dengan kutipan pasal dan penjelasan.

**Edge cases:**
- Dokumen dengan tabel atau annex: extract sebagai gambar, gunakan multi-modal LLM (GPT-4V) jika perlu.
- Bahasa campuran Indonesia-Inggris: prompt instruction untuk konsisten.
- Tidak ada pasal yang relevan: LLM harus bilang "Saya tidak menemukan informasi itu dalam kontrak."

## 7. Trade-offs & Alternatives
| Alternatif | Kelebihan | Kekurangan |
|------------|-----------|-------------|
| Kirim ke jasa review hukum manual | Akurat sekali | Mahal, lambat |
| Gunakan ChatGPT langsung (tanpa RAG) | Mudah | Halusinasi, tidak bisa refer ke dokumen spesifik |

**Dipilih RAG** karena memberikan jawaban berbasis dokumen aktual, bisa di-audit.

## 8. Risks & Mitigation
| Risiko | Mitigasi |
|--------|-----------|
| LLM memberikan saran hukum yang salah | Disclaimer di UI: "Ini bukan nasihat hukum profesional. Konsultasikan dengan lawyer untuk keputusan final." |
| OCR gagal pada scan buram | Integrasi dengan layanan OCR premium (Google Document AI) |
| Dokumen rahasia bocor | Enkripsi di rest, self-hosted LLM (Llama 3) agar data tidak keluar |

## 9. Metrics / Evaluation
- **Detection accuracy red flags**: Precision 90%, Recall 85% (test set 100 kontrak berlabel).
- **Answer relevance**: RAGAS score (faithfulness, answer relevancy) > 0.85.
- **User time saving**: survey user, rata-rata review time turun 80%.
- **Adoption rate**: >50% user yang coba akan menggunakan lagi.

## 10. Implementation Plan
- [ ] Minggu 1: Setup Qdrant, legal-BERT embedding, chunking pipeline
- [ ] Minggu 2: Implement risk detection (rule + similarity)
- [ ] Minggu 3: Integrasi LLM (Llama 3 70B via Together.ai atau lokal)
- [ ] Minggu 4: Buat frontend sederhana (upload & chat)
- [ ] Minggu 5: Testing dengan 50 kontrak riil (dari klien fiktif)
- [ ] Minggu 6: Deployment dengan Docker + dokumentasi API

**Tech stack:** Python, FastAPI, Qdrant, sentence-transformers, Llama 3 (via Groq/Together), React (frontend ringan).
**Kompleksitas:** Medium-Hard (karena perlu legal domain knowledge)
**Role:** AI Engineer (2), Backend (1), Frontend (1), Legal Consultant (part-time untuk label data)
# identitas
