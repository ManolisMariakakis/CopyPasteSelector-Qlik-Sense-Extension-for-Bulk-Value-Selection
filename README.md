# CopyPasteSelector – Qlik Sense Extension for Bulk Value Selection

A lightweight and efficient **Qlik Sense extension** that enables bulk selection of values (SKUs, IDs, or codes) in **any field** of your app — with a dynamically selectable field via dropdown.

---

## 🔍 Overview

This extension provides a **textarea** where users can **paste or type multiple values** (one per line).  
When clicking the **“Select”** button, the extension searches for these values in the chosen field and performs the selection automatically in the current state.

Users can also:
- Select the **target field dynamically** from a dropdown  
- Clear the input form  
- Clear selections in the chosen field  
- Configure the **available field list** directly from the **Edit Sheet** property panel  

---

## ⚙️ Features

✅ Bulk select values (one per line, or separated by `,` or `;`)  
✅ Dynamic field selection from dropdown  
✅ Configurable field list via property panel  
✅ Clear input or clear selections in field  
✅ Works with Alternate States  
✅ 100% native — uses only Qlik API / Enigma (no dependencies)  

---

## 🧩 Installation

1. **Download the ZIP file** of the extension from GitHub.  
2. In **Qlik Sense Enterprise for Windows**, go to the **QMC (Qlik Management Console)**.  
3. Open the **Extensions** section.  
4. Click **Import**, then select the downloaded ZIP file (`CopyPasteSelector.zip`).  
5. Once imported, open your Qlik Sense app and add **CopyPasteSelector** to any sheet.

> 🧪 Tested on **Qlik Sense Enterprise for Windows – May 2025 release**.

---

## 🧰 Configuration

In **Edit Sheet → Properties** of the object:

- **Field list (comma-separated):**  
  Provide the field names to appear in the dropdown, separated by commas.  
  Example:
  ```
  SKU_CODE,CUSTOMER_CODE,STORE_CODE
  ```

- **Default field:**  
  (Optional) Specify the default field name shown in the dropdown when the extension loads.

---

## 🧠 Usage

1. Select the desired field from the dropdown.  
2. Paste your values (one per line) into the textbox.  
3. Click **Select** to apply the selections.  
4. Use **Clear Form** or **Clear Field** as needed.

---

## 🛠️ Tech Notes

- Built with the standard `qlik` and `enigmaModel` APIs.  
- No external dependencies.  
- Compatible with Qlik Sense Desktop and Qlik Sense Enterprise.  
- Tested with Qlik Sense June 2025 release.

---

## 📄 License

GPL-3.0 license © 2025  
Developed by **Manolis Mariakakis** — “Bulk Value Selection” Extension

---

## 📜 Full JavaScript Source (`CopyPasteDynamicField.js`)

```javascript
define(["qlik"], function (qlik) {
  "use strict";

  var FIELD_NAME = "ΚΩΔ_ΕΙΔΟΥΣ"; 
  var PAGE_SIZE = 5000;

  const DEFAULT_OPTIONS_CSV = "ΚΩΔ_ΕΙΔΟΥΣ,ΚΩΔ_ΠΡΟΜ_ΕΙΔΟΥΣ,ΚΩΔ_ΚΑΤΑΣΤΗΜΑΤΟΣ,ΚΩΔ_ΑΛΥΣΙΔΑΣ";

  function getStateName(ctx) {
    try {
      return (ctx.backendApi &&
              ctx.backendApi.model &&
              ctx.backendApi.model.layout &&
              ctx.backendApi.model.layout.qStateName) || "$";
    } catch (e) {
      return "$";
    }
  }

  function sanitize(s) {
    return (s || "").replace(/[\u00A0\u2007\u202F]/g, " ").trim();
  }

  function parseWanted(raw) {
    if (!raw) return [];
    const out = new Set();
    String(raw)
      .split(/\r?\n/)
      .forEach(line => {
        line = sanitize(line);
        line.split(/[;,]/).forEach(part => {
          let v = sanitize(part)
            .replace(/^[“”"'`]|[“”"'`]$/g, "")
            .normalize("NFC");
          if (v) out.add(v);
        });
      });
    return Array.from(out);
  }

  function parseOptionsCsv(csvString) {
    const csv = (csvString || "").trim();
    const base = csv ? csv : DEFAULT_OPTIONS_CSV;
    return base
      .split(",")
      .map(sanitize)
      .filter(Boolean);
  }

  function createUI($element, currentField, optionsArr) {
    const el = $element[0];
    el.innerHTML = "";

    const wrap = document.createElement("div");
    wrap.style.font = "13px system-ui,-apple-system,Segoe UI,Roboto,Arial,sans-serif";

    const optsHtml = optionsArr
      .map(v => `<option value="${v}" ${v===currentField?"selected":""}>${v}</option>`)
      .join("");

    wrap.innerHTML = `
      <div style="display:flex;align-items:center;gap:8px;margin-bottom:6px;">
        <div style="font-weight:700;">Bulk Selection → Field</div>
        <select class="cp-field"
          style="padding:4px 8px;border:1px solid #ccc;border-radius:6px;background:#fff;cursor:pointer;">
          ${optsHtml}
        </select>
      </div>

      <div style="opacity:.8;margin-bottom:8px;">
        1. Enter one value per line or copy/paste your codes.<br>
        2. Click “Select”.
      </div>

      <textarea class="cp-text"
        style="min-height:260px;width:300px;padding:8px;border:1px solid #ccc;border-radius:8px;
               resize:vertical;font-family:monospace;"
        placeholder="e.g.\n14202121\n10001007"></textarea>

      <div style="display:flex;gap:8px;margin-top:8px;">
        <button class="cp-go"
          style="padding:6px 12px;border:1px solid #ccc;border-radius:6px;background:#f5f5f5;cursor:pointer;">
          Select
        </button>
      </div>
      <div style="display:flex;gap:8px;margin-top:8px;">
        <button class="cp-clear-input"
          style="padding:6px 12px;border:1px solid #ccc;border-radius:6px;background:#f5f5f5;cursor:pointer;">
          Clear Input
        </button>
      </div>
      <div style="display:flex;gap:8px;margin-top:8px;">
        <button class="cp-clear-field"
          style="padding:6px 12px;border:1px solid #ccc;border-radius:6px;background:#f5f5f5;cursor:pointer;">
          Clear Field <span class="cp-field-name">${currentField}</span>
        </button>
      </div>
      <div class="cp-status" style="margin-top:8px;min-height:18px;"></div>
    `;

    el.appendChild(wrap);

    return {
      fieldSel : wrap.querySelector(".cp-field"),
      fieldNameSpan : wrap.querySelector(".cp-field-name"),
      textarea : wrap.querySelector(".cp-text"),
      btnGo    : wrap.querySelector(".cp-go"),
      btnClrIn : wrap.querySelector(".cp-clear-input"),
      btnClrFld: wrap.querySelector(".cp-clear-field"),
      status   : wrap.querySelector(".cp-status")
    };
  }

  async function collectElemNumbers(listModel, totalCount, wantedSet) {
    const path = "/qListObjectDef";
    const elemNos = [];
    let top = 0;

    while (top < totalCount) {
      const height = Math.min(PAGE_SIZE, totalCount - top);
      const pages = await listModel.getListObjectData(path, [
        { qTop: top, qLeft: 0, qWidth: 1, qHeight: height }
      ]);
      const page = pages && pages[0];
      if (!page || !page.qMatrix) break;

      for (const row of page.qMatrix) {
        const cell = row[0];
        const txt = (cell.qText || "").trim();

        if (wantedSet.has(txt)) {
          elemNos.push(cell.qElemNumber);
        } else if (cell.qNum !== undefined && !isNaN(cell.qNum)) {
          const asIntStr = String(Math.trunc(cell.qNum));
          if (wantedSet.has(asIntStr)) elemNos.push(cell.qElemNumber);
        }
      }

      top += height;
      if (elemNos.length >= wantedSet.size) break;
    }
    return elemNos;
  }

  return {
    // Property panel: provide CSV list of fields
    definition: {
      type: "items",
      component: "accordion",
      items: {
        settings: {
          uses: "settings",
          items: {
            fieldsCfg: {
              label: "Field Dropdown Settings",
              type: "items",
              items: {
                fieldOptions: {
                  ref: "props.fieldOptions",
                  label: "Field list (comma-separated)",
                  type: "string",
                  expression: "optional",
                  defaultValue: DEFAULT_OPTIONS_CSV
                },
                defaultField: {
                  ref: "props.defaultField",
                  label: "Default field",
                  type: "string",
                  expression: "optional",
                  defaultValue: FIELD_NAME
                }
              }
            }
          }
        }
      }
    },

    paint: function ($element) {
      const layout = this.backendApi && this.backendApi.model && this.backendApi.model.layout;

      const optionsArr = parseOptionsCsv(layout && layout.props && layout.props.fieldOptions);
      let currentField = (layout && layout.props && layout.props.defaultField) || FIELD_NAME;
      if (!optionsArr.includes(currentField)) currentField = optionsArr[0] || FIELD_NAME;

      const ui = createUI($element, currentField, optionsArr);
      const stateName = getStateName(this);

      const app = qlik.currApp(this);
      let fieldHandle = app.field(currentField, stateName);

      const appModel  = this.backendApi.model && this.backendApi.model.enigmaModel;
      const enigmaApp = appModel && appModel.app;

      function setStatus(msg) { ui.status.textContent = msg || ""; }

      // change field from dropdown
      ui.fieldSel.addEventListener("change", (e) => {
        currentField = e.target.value || FIELD_NAME;
        ui.fieldNameSpan.textContent = currentField;
        fieldHandle = app.field(currentField, stateName);
        setStatus(`Field: ${currentField}`);
      });

      ui.btnClrIn.addEventListener("click", () => {
        ui.textarea.value = "";
        setStatus("");
      });

      ui.btnClrFld.addEventListener("click", async () => {
        try {
          setStatus(`Clearing selections for field ${currentField}...`);
          await fieldHandle.clear();
          setStatus("✅ Field selections cleared.");
        } catch (e) {
          console.error("[CopyPaste] clear field ERROR:", e);
          setStatus("❌ Error while clearing field (see Console).");
        }
      });

      ui.btnGo.addEventListener("click", async () => {
        const wanted = parseWanted(ui.textarea.value);
        if (!wanted.length) {
          setStatus("⚠️ No values found in input.");
          return;
        }
        if (!enigmaApp) {
          console.error("[CopyPaste] ERROR: enigmaApp not available");
          setStatus("❌ Error: enigma app not available.");
          return;
        }

        setStatus(`Searching and selecting ${wanted.length} values... (state ${stateName}, field ${currentField})`);
        let listModel = null;

        try {
          const def = {
            qInfo: { qType: "cp_dynamic_list" },
            qStateName: stateName,
            qListObjectDef: {
              qDef: { qFieldDefs: [currentField] },
              qShowAlternatives: true,
              qInitialDataFetch: [{ qTop: 0, qLeft: 0, qHeight: 0, qWidth: 1 }]
            }
          };

          listModel = await enigmaApp.createSessionObject(def);
          const layout = await listModel.getLayout();

          const total =
            (layout &&
             layout.qListObject &&
             layout.qListObject.qSize &&
             layout.qListObject.qSize.qcy) || 0;

          if (total === 0) {
            setStatus("⚠️ Field appears empty or inaccessible.");
            return;
          }

          const elemNos = await collectElemNumbers(listModel, total, new Set(wanted));

          if (!elemNos.length) {
            setStatus("⚠️ None of the values were found in the field.");
            return;
          }

          const ok = await listModel.selectListObjectValues("/qListObjectDef", elemNos, false, false);
          setStatus(ok ? `✅ Selection completed (${elemNos.length} values).`
                       : "⚠️ Selection not confirmed (locks or soft lock?).");
        } catch (err) {
          console.error("ERROR:", err);
          setStatus("❌ Error (see Console).");
        } finally {
          try {
            if (listModel && listModel.id) {
              await enigmaApp.destroySessionObject(listModel.id);
            }
          } catch (_) {}
        }
      });

      return qlik.Promise.resolve();
    },

    support: { snapshot: false, export: false, exportData: false }
  };
});

```

