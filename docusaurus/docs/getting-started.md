// Darwaza-Khidki Estimator - Single-file React web app (App.jsx) // Usage: paste this into src/App.jsx of a Create React App or Vite React project. // No extra npm packages required. Uses localStorage for persistence.

import React, { useState, useEffect } from 'react';

const STORAGE_KEY = '@quotes_list_v1_web';

export default function App() { const [lang, setLang] = useState('hi'); // 'hi' or 'en' const [type, setType] = useState('darwaza'); const [height, setHeight] = useState(''); const [width, setWidth] = useState(''); const [rate, setRate] = useState('1500'); const [labour, setLabour] = useState('500'); const [quotes, setQuotes] = useState([]);

useEffect(() => { loadQuotes(); }, []);

const loadQuotes = () => { try { const json = localStorage.getItem(STORAGE_KEY); if (json) setQuotes(JSON.parse(json)); } catch (e) { console.warn('Load failed', e); } };

const saveQuotes = (next) => { try { localStorage.setItem(STORAGE_KEY, JSON.stringify(next)); setQuotes(next); } catch (e) { console.warn('Save failed', e); } };

const parseNumber = (v) => { const n = Number(String(v).replace(/,/g, '')); return Number.isFinite(n) ? n : 0; };

const estimate = () => { const h = parseNumber(height) / 100; // meters const w = parseNumber(width) / 100; // meters const area = h * w; const materialRate = parseNumber(rate); const material = Number((area * materialRate).toFixed(2)); const labourCost = parseNumber(labour); const gst = Number((0.18 * (material + labourCost)).toFixed(2)); const total = Number((material + labourCost + gst).toFixed(2));

return { area: Number(area.toFixed(3)), material, labour: labourCost, gst, total };

};

const handleAddQuote = () => { if (!height || !width) { alert(lang === 'hi' ? 'ऊँचाई और चौड़ाई डालें' : 'Enter height and width'); return; } const est = estimate(); const item = { id: Date.now().toString(), type, height, width, rate, labour, ...est, createdAt: new Date().toISOString(), }; const next = [item, ...quotes]; saveQuotes(next); setHeight(''); setWidth(''); alert((lang === 'hi' ? 'अनुमान जोड़ा\n' : 'Estimate saved\n') + (lang==='hi'?'कुल: ':'Total: ') + '₹' + item.total); };

const handleDelete = (id) => { if (!window.confirm(lang==='hi'?'क्या आप हटाना चाहते हैं?':'Delete this quote?')) return; const next = quotes.filter((q) => q.id !== id); saveQuotes(next); };

const handleShare = async (q) => { const text = ${lang==='hi'?'प्रकार':'Type'}: ${q.type}\n${lang==='hi'?'आकार':'Size'}: ${q.height}x${q.width} cm\n${lang==='hi'?'क्षेत्रफल':'Area'}: ${q.area} m²\n${lang==='hi'?'सामग्री':'Material'}: ₹${q.material}\n${lang==='hi'?'श्रम':'Labour'}: ₹${q.labour}\nGST: ₹${q.gst}\n${lang==='hi'?'कुल':'Total'}: ₹${q.total}; if (navigator.share) { try { await navigator.share({ title: lang==='hi'?'अनुमान शेयर करें':'Share estimate', text }); } catch (e) { console.warn('Share failed', e); } } else { // fallback: copy to clipboard try { await navigator.clipboard.writeText(text); alert(lang==='hi'?'टेक्स्ट क्लिपबोर्ड पर कॉपी हुआ':'Text copied to clipboard'); } catch (e) { // show text in a prompt as last resort window.prompt(lang==='hi'?'यहाँ कॉपी करें:':'Copy this text:', text); } } };

const clearAll = () => { if (!window.confirm(lang==='hi'?'सभी रिकॉर्ड हटाएँ?':'Remove all saved quotes?')) return; saveQuotes([]); };

const downloadCSV = () => { if (!quotes.length) { alert(lang==='hi'?'कोई रिकॉर्ड नहीं':'No records'); return; } const header = ['type','height_cm','width_cm','area_m2','material','labour','gst','total','createdAt']; const rows = quotes.map(q => [q.type,q.height,q.width,q.area,q.material,q.labour,q.gst,q.total,q.createdAt]); const csv = [header, ...rows].map(r => r.map(String).map(v=>"${v.replace(/"/g,'""')}").join(',')).join('\n'); const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' }); const url = URL.createObjectURL(blob); const a = document.createElement('a'); a.href = url; a.download = 'estimates.csv'; a.click(); URL.revokeObjectURL(url); };

const t = (h, e) => (lang === 'hi' ? h : e);

return ( <div style={styles.app}> <div style={styles.card}> <div style={styles.header}> <h1 style={{margin:0}}>{t('दरवाज़ा/खिड़की अनुमान','Door/Window Estimator')}</h1> <div> <button style={styles.langBtn} onClick={() => setLang(lang==='hi'?'en':'hi')}>{lang==='hi'?'EN':'हिंदी'}</button> </div> </div>

<div style={styles.form}>
      <label style={styles.label}>{t('प्रकार','Type')}</label>
      <select value={type} onChange={e=>setType(e.target.value)} style={styles.select}>
        <option value="darwaza">{t('दरवाज़ा','Door')}</option>
        <option value="khidki">{t('खिड़की','Window')}</option>
        <option value="sheesha">{t('शीशा','Glass')}</option>
      </select>

      <div style={styles.row}>
        <input placeholder={t('ऊँचाई (cm)','Height (cm)')} value={height} onChange={e=>setHeight(e.target.value)} style={styles.input} />
        <input placeholder={t('चौड़ाई (cm)','Width (cm)')} value={width} onChange={e=>setWidth(e.target.value)} style={styles.input} />
      </div>

      <div style={styles.row}>
        <input placeholder={t('सामग्री दर प्रति m²','Material rate per m²')} value={rate} onChange={e=>setRate(e.target.value)} style={styles.input} />
        <input placeholder={t('श्रम (₹)','Labour (₹)')} value={labour} onChange={e=>setLabour(e.target.value)} style={styles.input} />
      </div>

      <div style={{display:'flex',gap:8}}>
        <button style={styles.primary} onClick={handleAddQuote}>{t('अनुमान जोड़ें','Save estimate')}</button>
        <button style={styles.secondary} onClick={()=>alert(JSON.stringify(estimate()))}>{t('तुरंत देखें','Quick estimate')}</button>
      </div>
    </div>

    <div style={{display:'flex',justifyContent:'space-between',alignItems:'center',marginTop:12}}>
      <h3 style={{margin:0}}>{t('बचाए गए अनुमान','Saved estimates')}</h3>
      <div style={{display:'flex',gap:8}}>
        <button onClick={downloadCSV} style={{...styles.link}}>{t('डाउनलोड CSV','Download CSV')}</button>
        <button onClick={clearAll} style={{...styles.link, color:'#ff4d4d'}}>{t('साफ़ करें','Clear')}</button>
      </div>
    </div>

    <div style={{marginTop:10}}>
      {quotes.length===0 ? (
        <p style={{textAlign:'center',color:'#666'}}>{t('कोई रिकॉर्ड नहीं','No records yet')}</p>
      ) : (
        quotes.map(q => (
          <div key={q.id} style={styles.rowCard}>
            <div>
              <div style={{fontWeight:700}}>{q.type} — {q.height} x {q.width} cm</div>
              <div style={{color:'#666',fontSize:13}}>Area: {q.area} m² • Total: ₹{q.total}</div>
              <div style={{color:'#999',fontSize:12}}>{new Date(q.createdAt).toLocaleString()}</div>
            </div>
            <div style={{display:'flex',flexDirection:'column',gap:6}}>
              <button style={styles.smallBtn} onClick={()=>handleShare(q)}>{t('शेयर','Share')}</button>
              <button style={{...styles.smallBtn,background:'#ff6b6b'}} onClick={()=>handleDelete(q.id)}>{t('हटाएँ','Delete')}</button>
            </div>
          </div>
        ))
      )}
    </div>

    <div style={{textAlign:'center',marginTop:18,color:'#999'}}>Made with ❤️</div>
  </div>

  {/* basic styles inlined so the file is self-contained */}
  <style>{`body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial; background:#f5f7fb; }
  `}</style>
</div>

); }

const styles = { app: { padding: 16, minHeight:'100vh' }, card: { maxWidth:900, margin:'0 auto', background:'#fff', padding:16, borderRadius:10, boxShadow:'0 6px 18px rgba(0,0,0,0.06)' }, header: { display:'flex', justifyContent:'space-between', alignItems:'center' }, langBtn: { padding:'8px 10px', borderRadius:6, background:'#eee', border:'none', cursor:'pointer' }, form: { marginTop:12, background:'#fafafa', padding:12, borderRadius:8 }, label: { fontWeight:700, marginBottom:6, display:'block' }, select: { width:'100%', padding:8, borderRadius:6, border:'1px solid #ddd', marginBottom:8 }, row: { display:'flex', gap:8, marginBottom:8 }, input: { flex:1, padding:8, borderRadius:6, border:'1px solid #ddd' }, primary: { flex:1, padding:10, borderRadius:8, background:'#2b8aef', color:'#fff', border:'none', cursor:'pointer' }, secondary: { flex:1, padding:10, borderRadius:8, border:'1px solid #2b8aef', color:'#2b8aef', background:'transparent', cursor:'pointer' }, link: { background:'transparent', border:'none', cursor:'pointer', color:'#2b8aef', padding:6 }, rowCard: { display:'flex', justifyContent:'space-between', alignItems:'center', padding:10, borderRadius:8, background:'#fff', border:'1px solid #eee', marginTop:10 }, smallBtn: { padding:8, borderRadius:6, background:'#2b8aef', color:'#fff', border:'none', cursor:'pointer' } };

