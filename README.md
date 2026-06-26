# MyMedfit BRAVO PRO · ANCHOR Integrated

## Complete Patient Assessment & Verification System

---

## 📋 Overview

MyMedfit BRAVO PRO is an integrated patient assessment tool that combines:

- **Comprehensive patient assessment** with laboratory values and functional scores
- **ANCHOR biometric verification** (liveness detection + optical heart rate)
- **KYC caregiver identification** (name, license, issuing authority)
- **Signed session token generation** for verifier compatibility
- **Local token vault** for secure token storage and management

**All processing runs locally in the browser - no data is sent to any cloud server.**

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                     MyMedfit BRAVO PRO                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐         │
│  │   ANCHOR     │    │  Patient     │    │   Token      │         │
│  │  Biometric   │    │ Assessment   │    │   Vault      │         │
│  │  Verification│    │              │    │              │         │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘         │
│         │                   │                   │                  │
│         ▼                   ▼                   ▼                  │
│  ┌─────────────────────────────────────────────────────┐          │
│  │              ANCHOR Session Token                    │          │
│  │  { liveness ✓, HR, KYC, assessment, signature }    │          │
│  └─────────────────────────────────────────────────────┘          │
│                              │                                      │
│                              ▼                                      │
│                    ┌─────────────────┐                             │
│                    │  Verifier Tool  │                             │
│                    │  (manual share) │                             │
│                    └─────────────────┘                             │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Components

### 1. ANCHOR Biometric Verification

| Component | Description | Status Indicators |
|-----------|-------------|-------------------|
| **Liveness Detection** | 4-step guided head movement test | ✅ Completed / ⏳ Pending |
| **Optical HR (rPPG)** | Camera-based heart rate measurement | Green (good) / Yellow (low) / Red (none) |
| **Face Detection** | Automatic ROI tracking for HR | ✓ Active / ✗ Not available |

**Liveness Steps:**
1. Look straight into the camera (25%)
2. Turn head slowly to the LEFT (50%)
3. Turn head slowly to the RIGHT (75%)
4. Nod head UP and DOWN (100%)

### 2. KYC - Caregiver Identity

| Field | Description | Required |
|-------|-------------|----------|
| Full Name | Caregiver's complete name | ✅ Yes |
| License/ID | Professional license number | ✅ Yes |
| Issuing Authority | Organization that issued the license | ✅ Yes |
| Encounter Reference | Optional reference number | ❌ No |
| Device ID | Automatically generated per device | ✅ Yes |

### 3. Patient Assessment & Labs

**Functional Scores:**
- Pain (NRS 0-10)
- Mobility (0-10)
- Fatigue (0-10)
- Global Function (0-10)

**Vital Signs:**
- Heart Rate (bpm)
- SBP / DBP (mmHg)

**Metabolic:**
- Fasting Glucose (mmol/L)
- HbA1c (%)

**Lipids:**
- Total Cholesterol (mmol/L)
- LDL (mmol/L)
- HDL (mmol/L)
- Triglycerides (mmol/L)

**Renal:**
- Creatinine (μmol/L)
- eGFR (mL/min/1.73m²)

**Nutrition/Anaemia:**
- Haemoglobin (g/dL)
- Albumin (g/L)

**Inflammation:**
- CRP (mg/L)

**Cardiac:**
- Ejection Fraction (%)
- BNP/NT‑proBNP (pg/mL) – Acute & Palliative only

**Respiratory:**
- FEV1 (L)
- FVC (L)

**Profile-Specific:**
- BMD T‑Score (Preventive)
- Exercise (Preventive)
- Comfort Score (Palliative)

### 4. ANCHOR Session Token

The token is a signed JWT containing:

```javascript
{
  // ANCHOR verification data
  anchor: {
    liveness_completed: boolean,
    liveness_duration_sec: number,
    hr_bpm: number,
    hr_quality: number,  // 0-100%
    verified_at: ISO timestamp,
    device_id: string
  },
  
  // KYC data
  kyc: {
    caregiver_name: string,
    caregiver_license: string,
    issuing_authority: string,
    encounter_ref: string
  },
  
  // Assessment data
  assessment: {
    functionality_index: number,  // 0-100
    pain: number,
    mobility: number,
    fatigue: number,
    function: number,
    profile: string  // acute | preventive | palliative
  },
  
  // Token metadata
  iat: Unix timestamp,
  exp: Unix timestamp,  // 10 minutes validity
  jti: string,  // Unique token ID
  version: "anchor-token-1.0.0"
}
```

**Token Format:**
```
{header}.{payload}.{signature}
```

**Signature:** HMAC-SHA256 with shared secret key `local-offline-signature`

### 5. Token Vault

- **Local Storage Key:** `medfitBravo_tokenVault`
- **Storage Format:** Array of token objects
- **Each Token:** { token, date, caregiver, device, verified }
- **Actions:** Copy, Delete, Export JSON, Verify

---

## 🚀 Installation & Usage

### Quick Start

1. **Clone or download** the HTML file
2. **Open in a modern browser** (Chrome, Firefox, Edge, Safari)
3. **No server required** - runs entirely client-side

### Step-by-Step Workflow

#### Step 1: Start Camera
```
Click "Start Camera" → Allow camera permissions
```

#### Step 2: Run Liveness
```
Click "Run Liveness" → Follow the 4 guided movements
Wait for "Liveness completed ✓"
```

#### Step 3: Measure Heart Rate
```
Click "Start HR" → Stay still for 8-15 seconds
Wait for HR reading to appear (green dot = good quality)
Click "Stop HR" when finished
```

#### Step 4: Enter KYC Data
```
Fill in:
- Caregiver Name
- License/ID
- Issuing Authority
- Encounter Reference (optional)
```

#### Step 5: Generate ANCHOR Token
```
Click "Generate Token" → Token appears in the text area
Token includes: Liveness ✓ + HR ✓ + KYC ✓
```

#### Step 6: Save to Vault (Optional)
```
Click "Save to Vault" → Token stored locally
View, copy, or delete tokens from vault
```

#### Step 7: Share with Verifier
```
Click "Copy" → Token copied to clipboard
Manually share with ANCHOR verifier tool
```

#### Step 8: Patient Assessment
```
Enter patient demographics
Enter laboratory values
Set care goals (Pain, Mobility, Energy)
Click "Save" to record the encounter
```

#### Step 9: Generate PDF Report
```
Click "PDF Report" → Download comprehensive report
Includes: Patient data, ANCHOR verification, assessment, trends
```

---

## 📁 Data Storage

All data is stored locally in the browser's `localStorage`:

| Storage Key | Content | Purpose |
|-------------|---------|---------|
| `encounter_device_id` | Unique device identifier | Device binding |
| `medfitBravo_encounters` | Patient assessment history | Encounter records |
| `medfitBravo_tokenVault` | Generated session tokens | Token vault |

**⚠️ Important:** Data is tied to the specific device and browser. Clearing browser data or using a different device will not have access to previously stored data.

---

## 🔐 Security Features

| Feature | Description |
|---------|-------------|
| **Local Processing** | All biometric and assessment data stays in the browser |
| **No Cloud Storage** | No external API calls for sensitive data |
| **Signed Tokens** | JWT with shared secret key for verifier compatibility |
| **Device Binding** | Each token is bound to a specific device ID |
| **Token Expiration** | Tokens expire after 10 minutes |
| **Manual Sharing** | Tokens are shared manually - no automatic transmission |

---

## 🧪 Testing & Verification

### Test the Token with Verifier

1. **Generate a token** using the ANCHOR tool
2. **Copy the token** to clipboard
3. **Paste into verifier tool** (if available)
4. **Expected result:** Token signature is validated, payload is decoded

### Manual Token Verification

The token can be manually verified by:

1. **Split the token** at the periods (`.`)
2. **Decode the header** (Base64Url)
3. **Decode the payload** (Base64Url)
4. **Check the signature** against `local-offline-signature`
5. **Verify expiration** (exp timestamp > current time)

---

## 📊 PDF Report Contents

The generated PDF report includes:

1. **Patient Information**
   - Gender, Age, BMI, Profile, Caregiver name, Device ID

2. **ANCHOR Verification**
   - Liveness status, HR BPM, HR quality, Verification status

3. **Current Readings**
   - All laboratory values and functional scores

4. **Functionality Index**
   - Score (0-100) with risk level indicator

5. **ICF Domains**
   - Mapped to International Classification of Functioning categories

6. **Care Tips**
   - Evidence-based recommendations based on patient data

7. **Trend Summary**
   - Last 5 encounters with key metrics

---

## 🌐 Browser Compatibility

| Browser | Supported | Notes |
|---------|-----------|-------|
| Chrome 80+ | ✅ Yes | Full support |
| Firefox 75+ | ✅ Yes | Full support |
| Edge 80+ | ✅ Yes | Full support |
| Safari 13+ | ✅ Yes | Camera access required |
| Mobile Safari | ✅ Yes | iOS 14+ |
| Android Chrome | ✅ Yes | Full support |

**Requirements:**
- Modern browser with WebRTC support
- Camera access for liveness and HR
- JavaScript enabled
- LocalStorage enabled

---

## 🔧 Configuration

### Shared Secret Key

```javascript
const TOKEN_KEY_STRING = "local-offline-signature";
```

This key is shared between the ANCHOR tool and the verifier. Change it to a custom value if needed.

### Token Expiration

```javascript
const expSec = nowSec + 600;  // 10 minutes (600 seconds)
```

Modify this value to change token validity period.

### HR Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| HR_MIN_BPM | 45 | Minimum heart rate |
| HR_MAX_BPM | 160 | Maximum heart rate |
| HR_SAMPLE_INTERVAL_MS | 80 | Sampling interval |
| HR_MIN_WINDOW_SEC | 8 | Minimum window for estimation |
| HR_MAX_WINDOW_SEC | 25 | Maximum window for estimation |

---

## 📝 Glossary

| Term | Definition |
|------|------------|
| **ANCHOR** | Authentication & Non-repudiation with Camera-based Heart rate + Liveness |
| **rPPG** | Remote Photoplethysmography - measuring heart rate from video |
| **KYC** | Know Your Customer - identity verification |
| **JWT** | JSON Web Token - signed token format |
| **ICF** | International Classification of Functioning, Disability and Health |
| **NRS** | Numeric Rating Scale (0-10) |
| **POCT** | Point-of-Care Testing |

---

## 👨‍💻 Developer Notes

### Adding New Assessment Fields

1. Open the `FIELD_DEFS` object
2. Add new field definitions:

```javascript
{ id: 'new_field', label: 'Field Name', unit: 'unit', step: '0.1' }
```

### Modifying the Token Payload

1. Find the `generateToken()` function
2. Update the `payload` object structure
3. Ensure the verifier tool is updated accordingly

### Changing the UI Theme

The tool uses a dark theme by default. To customize:

1. Modify CSS variables in the `<style>` section
2. Update colors: `#0b1020`, `#151a30`, `#005792`, `#00b4d8`

---

## 📞 Contact & Support

**Developer:** Moustafa Mohammed Elsayed Elsayed Ali  
**Email:** mostischmidbauer@web.de  
**Email:** agentic@virtualcaresolution.de  
**Phone:** +49 1522 5321568  
**Address:** Kronwittener Straße 1, 84367 Tann, Germany  
**Website:** [virtualcaresolution.de](https://virtualcaresolution.de)

---

## ⚖️ Disclaimer

This tool is provided for informational and monitoring purposes only. It is **not a certified medical device**. The optical heart rate measurement and liveness detection are auxiliary signals and must not be used for:

- Medical diagnosis
- Treatment decisions
- Emergency situations
- Legal identification alone

All processing runs locally in the browser. Users are responsible for compliance with applicable data protection laws and institutional policies.

---

## 📄 License

This tool is intended for use within organizational policies and applicable law. No warranty is provided, express or implied.

---

## 🔄 Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-06-26 | Initial release with ANCHOR integration |

---

**Made with ❤️ for patient-centered care**
