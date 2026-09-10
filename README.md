# SonibaiKaKhakara

Single-file billing app for the Khakra shop — billing, products, history, ledger, WhatsApp bills, PDF/print bills, realtime sync.

## 🔐 Suraksha Seal — Bill Lock & Key (tamper-proof bills)

Every **Paid** and **Due/Unpaid** bill gets a unique cryptographic **Seal Code** the moment it is saved, printed, downloaded as PDF, or sent on WhatsApp.

**How it works (lock & key)**

- The app holds a secret **Lock** — a 256-bit key generated once per shop (`Settings → Bill Seal Lock`). It is **never printed or shared**.
- Each bill's **Seal Code (key)** = `HMAC-SHA256(lock, billNo | date | customer | items | subtotal | charges | discount | total | paid | due | status)`, encoded as 12 Crockford-Base32 chars (60-bit ≈ 1-in-10¹⁸ guess) + 1 typo-catching checksum char.
- Format: `SK-<LOCK>-P-XXXX-XXXX-XXXXX` for **Paid** (green key), `SK-<LOCK>-D-XXXX-XXXX-XXXXX` for **Due/Unpaid** (red key).
- Without the lock, a fake but valid-looking code cannot be forged; changing even ₹0.01, one quantity, or the status letter breaks the code.

**What the receiver sees**

- **Paid bills** — green diagonal `PAID` watermark, green `PAID IN FULL` stamp, green seal box with code + visual seal pattern.
- **Due/Unpaid bills** — soft-red paper tint, soft-red diagonal `DUE` watermark, red stamp showing the due amount, red seal box.
- Same seal on print, PDF download, and WhatsApp text (which also lists the sealed Total/Paid/Due).

**Verification (🔐 Verify tab)**

- Enter Bill No + the code from the paper (+ optionally the total printed on it) → **GENUINE / TAMPERED / FAKE** with reasons (wrong lock, status-letter mismatch, amount mismatch, checksum typo).
- A DUE code can never pass as PAID: recording a payment instantly issues a brand-new PAID code and retires the old one (old codes kept in bill `sealHistory`).
- The deterministic 5×5 visual seal pattern must match the one printed on the bill.
- Regenerating the lock keeps the previous lock for verifying already-shared bills (`ok-prev` verdict).

**Files**: everything lives in `index.html` (app + seal engine + pure-JS SHA-256/HMAC, works offline and on `file://`).
