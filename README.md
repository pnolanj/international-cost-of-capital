# Global Corporate Finance | International Cost of Capital & WACC Intelligence

An institutional-grade Streamlit web application (`app.py`) that retrieves live financial and macroeconomic data from **Aswath Damodaran (NYU Stern)**, the **OECD**, and **Trading Economics** to compute and compare a project's **Cost of Equity ($r_E$)**, **Pre-Tax and After-Tax Cost of Debt ($r_D$)**, and **Weighted Average Cost of Capital ($\text{WACC}$)** in **USD** versus the **local currency of any selected target country**.

---

## 1. Financial & Mathematical Framework

All rate inputs and outputs in the user interface are displayed as percentages (e.g., `6.50%`), while internal calculations operate in full floating-point decimal precision (`0.065`).

### 1.1 Shared Project Parameters
- **Unlevered Asset Beta ($\beta_U$)**: Retrieved from Damodaran's U.S. industry dataset (`betas.xls`) or manually overwritten.
- **Target Capital Structure**:
  - Target Debt Share: $w_D = \frac{D}{V} \in [0, 0.90]$
  - Target Equity Share: $w_E = \frac{E}{V} = 1 - w_D$
  - Debt-to-Equity Ratio: $\frac{D}{E} = \frac{w_D}{1 - w_D}$
- **Active Project Corporate Tax Rate ($\tau$)**: Users can toggle between the **U.S. Statutory Corporate Tax Rate ($\tau_{\text{US}}$)** and the **Selected Country Statutory Corporate Tax Rate ($\tau_{\text{FC}}$)** (or supply a manual override). The active tax rate $\tau$ is applied identically in both currencies.
- **Levered Equity Beta ($\beta_L$)**:
  - **Hamada Tax-Adjusted Convention (Default)**:
    $$\beta_L = \beta_U \times \left[1 + (1 - \tau) \times \frac{D}{E}\right]$$
  - **Unadjusted Re-Levering Convention** (available via sidebar methodology toggle):
    $$\beta_L = \beta_U \times \left[1 + \frac{D}{E}\right]$$

---

### 1.2 USD Cost of Capital Baseline
Using the U.S. 10-year Treasury yield ($r_{f,\text{USD}}$), the Damodaran `Baa2/BBB` synthetic corporate default spread for large non-financial firms ($s_{\text{USD}}$), and the mature market Equity Risk Premium ($\text{ERP}$):

- **USD Cost of Equity**:
  $$r_{E,\text{USD}} = r_{f,\text{USD}} + \beta_L \times \text{ERP}$$
- **USD Pre-Tax & After-Tax Cost of Debt**:
  $$r_{D,\text{USD}} = r_{f,\text{USD}} + s_{\text{USD}}, \qquad r_{D,\text{USD}}^{\text{after-tax}} = r_{D,\text{USD}} \times (1 - \tau)$$
- **USD Project WACC**:
  $$\text{WACC}_{\text{USD}} = w_E \times r_{E,\text{USD}} + w_D \times r_{D,\text{USD}} \times (1 - \tau)$$

---

### 1.3 Foreign Currency (FC) Cost of Capital via International Parity
Let $r_{f,\text{FC}}$ denote the selected country's 10-year government bond yield. Define the **International Fisher / Covered Interest Rate Parity scaling factor**:
$$\Pi = \frac{1 + r_{f,\text{FC}}}{1 + r_{f,\text{USD}}}$$

1. **Foreign Currency Cost of Equity ($r_{E,\text{FC}}$)**:
   $$1 + r_{E,\text{FC}} = (1 + r_{E,\text{USD}}) \times \frac{1 + r_{f,\text{FC}}}{1 + r_{f,\text{USD}}} \implies r_{E,\text{FC}} = (1 + r_{E,\text{USD}}) \times \Pi - 1$$

2. **Foreign Currency BBB Borrowing Spread ($s_{\text{FC}}$) & Cost of Debt ($r_{D,\text{FC}}$)**:
   - **When Foreign Spread Manual Override is OFF (`Auto-Calculated via Parity`)**:
     $$1 + r_{D,\text{FC}}^{\text{auto}} = (1 + r_{D,\text{USD}}) \times \Pi \implies s_{\text{FC}}^{\text{auto}} = r_{D,\text{FC}}^{\text{auto}} - r_{f,\text{FC}} = s_{\text{USD}} \times \frac{1 + r_{f,\text{FC}}}{1 + r_{f,\text{USD}}}$$
   - **When Foreign Spread Manual Override is ON (`Manual Override`)**:
     $$r_{D,\text{FC}} = r_{f,\text{FC}} + s_{\text{FC}}^{\text{manual}}$$
     *(Overriding $s_{\text{FC}}$ updates only $r_{D,\text{FC}}$ and $\text{WACC}_{\text{FC}}$, leaving all USD results and $r_{E,\text{FC}}$ completely unchanged).*

3. **Foreign Currency Project WACC ($\text{WACC}_{\text{FC}}$)**:
   $$\text{WACC}_{\text{FC}} = w_E \times r_{E,\text{FC}} + w_D \times r_{D,\text{FC}} \times (1 - \tau)$$

---

## 2. Live Data Sources & Two-Tier Resilience Architecture

Each external source is cached for 1 hour (`@st.cache_data(ttl=3600)`) and can be refreshed on demand via the **"🔄 Refresh Live Market Data"** button in the sidebar. If a live endpoint blocks cloud IPs or alters its HTML structure, the app automatically falls back to a comprehensive built-in **Last-Known-Good (LKG) snapshot** (94 Damodaran industries and 42 global economies) without raising an unhandled exception:

1. **Damodaran U.S. Industry Betas**: `https://pages.stern.nyu.edu/~adamodar/pc/datasets/betas.xls` (parsed via `pandas` + `xlrd`).
2. **Damodaran BBB USD Borrowing Spread**: `https://pages.stern.nyu.edu/~adamodar/New_Home_Page/datafile/ratings.html` (`Baa2/BBB` row for large non-financial service firms).
3. **OECD Statutory Corporate Income Tax Rates**: `https://www.oecd.org/en/publications/corporate-tax-statistics-2024_9c27d6e8-en/full-report/statutory-corporate-income-tax-rates_440f9286.html#chapter-d1e2342-08e74a33fb`.
4. **Trading Economics 10-Year Government Bond Yields**: `https://tradingeconomics.com/bonds`.

---

## 3. Verification & Diagnostic Checklist

Open **Tab 3 (`Methodology, Assumptions & Diagnostic Checks`)** in the app to view the live automated verification suite and 6-decimal-place arithmetic plug-in proof:

1. **Equal Government Bond Yields Check**: When $r_{f,\text{FC}} = r_{f,\text{USD}}$ and the foreign-currency spread override is OFF, $\Pi = 1.000000$, confirming $r_{E,\text{FC}} = r_{E,\text{USD}}$, $s_{\text{FC}} = s_{\text{USD}}$, $r_{D,\text{FC}} = r_{D,\text{USD}}$, and $\text{WACC}_{\text{FC}} = \text{WACC}_{\text{USD}}$.
2. **Zero Debt Check**: Setting Target Debt Share $D/V = 0\%$ yields $\frac{D}{E} = 0$, $\beta_L = \beta_U$, $\text{WACC}_{\text{USD}} = r_{E,\text{USD}}$, and $\text{WACC}_{\text{FC}} = r_{E,\text{FC}}$.
3. **Foreign Spread Override Isolation**: Toggling or changing the manual foreign BBB spread $s_{\text{FC}}$ modifies only $r_{D,\text{FC}}$ and $\text{WACC}_{\text{FC}}$, leaving $r_{E,\text{USD}}$, $r_{D,\text{USD}}$, and $\text{WACC}_{\text{USD}}$ 100% invariant.

---

## 4. Local Execution & Streamlit Community Cloud Deployment

### Run Locally (Python 3.12+)
```bash
pip install -r requirements.txt
streamlit run app.py
