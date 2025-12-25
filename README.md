import React, { useState, useRef, useEffect } from 'react';
import { 
  Database, 
  Table as TableIcon, 
  Plus, 
  Trash2, 
  Search, 
  ChevronRight, 
  CheckCircle2,
  Upload,
  UserPlus,
  ListPlus,
  SendHorizontal,
  Tag,
  Menu,
  AlertCircle,
  Printer,
  Calculator
} from 'lucide-react';

const App = () => {
  // --- 常數定義 ---
  const DEFAULT_BANK_ID = '0040314';
  const DEFAULT_NAME = '雲林國民小學';
  const DEFAULT_TYPE = '11'; 
  const DEFAULT_NOTE1 = '製表     出納        會計          校長';
  const DEFAULT_NOTE2 = ''; 
  const DEFAULT_NOTE3 = '連  絡  人:林足厚(電話:0921702706)統編:78959576';
  
  const NOTE_OPTIONS = ["薪資", "轉帳", "匯款", "扶幼補助款", "豐泰獎學金", "其他"];
  const PURPOSE_OPTIONS = ["員工薪資轉帳", "廠商貨款支付", "各項獎學金發放", "扶幼補助款發放", "其他"];

  const getTodayYYMMDDD = () => {
    const d = new Date();
    const y = (d.getFullYear() - 1911).toString().padStart(3, '0');
    const m = (d.getMonth() + 1).toString().padStart(2, '0');
    const day = d.getDate().toString().padStart(2, '0');
    return `${y}${m}${day}`;
  };

  // --- 狀態管理 ---
  const [viewMode, setViewMode] = useState('form_remitter');
  const [isMobileMenuOpen, setIsMobileMenuOpen] = useState(false);
  
  // 1. R_base (匯款人資料)
  const [dataListBase, setDataListBase] = useState([{ 
    id: 1, 
    bankId: DEFAULT_BANK_ID, 
    name: DEFAULT_NAME, 
    type: DEFAULT_TYPE, 
    note1: DEFAULT_NOTE1, 
    note2: DEFAULT_NOTE2, 
    note3: DEFAULT_NOTE3 
  }]);

  // 2. R_CUST (收款人資料) - 完整預載 R_CUST2.json 內容
  const [dataListCust, setDataListCust] = useState([
    { id: 1001, custCode: '0002', custName: '正凱食品有限公司', custAccount: '031001123811', bankAccount: '0040314' },
    { id: 1002, custCode: '0003', custName: '文吉瓦斯行-黃文吉', custAccount: '031001123739', bankAccount: '0040314' },
    { id: 1003, custCode: '0005', custName: '張進埤', custAccount: '00515140292970', bankAccount: '7000021' },
    { id: 1004, custCode: '0007', custName: '廖素如', custAccount: '031004311323', bankAccount: '0040314' },
    { id: 1005, custCode: '0092', custName: '玉贊成號林國回', custAccount: '040135932000', bankAccount: '0096204' },
    { id: 1006, custCode: '0093', custName: '連興蛋行陳銘宏', custAccount: '031001005957', bankAccount: '0040314' },
    { id: 1007, custCode: '0094', custName: '綠德光電股份有限公司', custAccount: '153100725827', bankAccount: '0081533' },
    { id: 1008, custCode: '0075', custName: '賴信嵐', custAccount: '03010080375279', bankAccount: '7000021' },
    { id: 1009, custCode: '0095', custName: '江國誠', custAccount: '00183211627', bankAccount: '0040180' }
  ]);

  // 3. R_DETL (匯款明細資料)
  const [dataListDetl, setDataListDetl] = useState([]);
  
  // 4. R_DAY (匯款批號資料)
  const [dataListDay, setDataListDay] = useState([{
    id: 1,
    remitDate: getTodayYYMMDDD(),
    batchNo: getTodayYYMMDDD().slice(1) + '0101',
    purpose: '員工薪資轉帳'
  }]);

  // 表單狀態
  const [tempDetailList, setTempDetailList] = useState([]);
  const [formRemitter, setFormRemitter] = useState({ ...dataListBase[0] });
  const [formRecipient, setFormRecipient] = useState({ custCode: '', custName: '', custAccount: '', bankAccount: '' });
  const [formDetail, setFormDetail] = useState({ custCode: '', custName: '', custAccount: '', bankAccount: '', amount: '', fee: '0', memo: '薪資' });
  const [formDay, setFormDay] = useState({ remitDate: getTodayYYMMDDD(), batchNo: getTodayYYMMDDD().slice(1) + '0101', purpose: '員工薪資轉帳' });

  const [custSearchTerm, setCustSearchTerm] = useState(''); 
  const [showCustSuggestions, setShowCustSuggestions] = useState(false);
  const [showSuccess, setShowSuccess] = useState(false);

  const handleShowSuccess = () => { setShowSuccess(true); setTimeout(() => setShowSuccess(false), 3000); };

  const selectSuggestion = (cust) => {
    setFormDetail({ 
      ...formDetail, 
      custCode: cust.custCode, 
      custName: cust.custName, 
      custAccount: cust.custAccount, 
      bankAccount: cust.bankAccount 
    });
    setCustSearchTerm(cust.custName);
    setShowCustSuggestions(false);
  };

  const getSuggestions = () => {
    if (!custSearchTerm) return [];
    return dataListCust.filter(c => c.custName.includes(custSearchTerm) || c.custCode.includes(custSearchTerm)).slice(0, 10);
  };

  const deleteRow = (id, listType) => {
    if (listType === 'cust') setDataListCust(prev => prev.filter(i => i.id !== id));
    else if (listType === 'temp') setTempDetailList(prev => prev.filter(i => i.id !== id));
    else if (listType === 'detl') setDataListDetl(prev => prev.filter(i => i.id !== id));
    else if (listType === 'base') setDataListBase(prev => prev.filter(i => i.id !== id));
    else if (listType === 'day') setDataListDay(prev => prev.filter(i => i.id !== id));
  };

  const handlePrint = () => {
    const printContent = document.getElementById('print-area').innerHTML;
    const printWindow = window.open('', '_blank');
    printWindow.document.open();
    printWindow.document.write(`
      <html>
        <head>
          <title>整批匯款明細列印</title>
          <script src="https://cdn.tailwindcss.com"></script>
          <style>
            @media print { @page { size: A4 landscape; margin: 0.5cm; } body { -webkit-print-color-adjust: exact; } }
            body { font-family: "Courier New", Courier, monospace; background-color: white; }
          </style>
        </head>
        <body class="p-4" onload="setTimeout(() => { window.print(); window.close(); }, 500)">
          <div class="print-container">${printContent}</div>
        </body>
      </html>
    `);
    printWindow.document.close();
  };

  const NavLinks = () => (
    <div className="flex flex-col space-y-1">
      <div className="px-4 py-2 text-[10px] font-bold text-slate-500 uppercase tracking-widest mt-4">基本資料維護</div>
      <button onClick={() => setViewMode('form_remitter')} className={`mx-2 flex items-center gap-3 px-4 py-3 text-sm rounded-xl transition-all ${viewMode === 'form_remitter' ? 'bg-blue-600 text-white' : 'text-slate-400 hover:bg-slate-800'}`}>
        <Database size={18} /> <span className="font-semibold">匯款人資料建檔</span>
      </button>
      <button onClick={() => setViewMode('form_recipient')} className={`mx-2 flex items-center gap-3 px-4 py-3 text-sm rounded-xl transition-all ${viewMode === 'form_recipient' ? 'bg-indigo-600 text-white' : 'text-slate-400 hover:bg-slate-800'}`}>
        <UserPlus size={18} /> <span className="font-semibold">收款人資料建檔</span>
      </button>
      <button onClick={() => setViewMode('form_day')} className={`mx-2 flex items-center gap-3 px-4 py-3 text-sm rounded-xl transition-all ${viewMode === 'form_day' ? 'bg-amber-600 text-white' : 'text-slate-400 hover:bg-slate-800'}`}>
        <Tag size={18} /> <span className="font-semibold">匯款批號輸入</span>
      </button>
      <div className="px-4 py-2 text-[10px] font-bold text-slate-500 uppercase tracking-widest mt-4 border-t border-slate-800 pt-4">帳務管理</div>
      <button onClick={() => setViewMode('form_detl')} className={`mx-2 flex items-center gap-3 px-4 py-3 text-sm rounded-xl transition-all ${viewMode === 'form_detl' ? 'bg-emerald-600 text-white' : 'text-slate-400 hover:bg-slate-800'}`}>
        <Calculator size={18} /> <span className="font-semibold">匯款明細輸入</span>
      </button>
      <button onClick={() => setViewMode('print_preview')} className={`mx-2 flex items-center gap-3 px-4 py-3 text-sm rounded-xl transition-all ${viewMode === 'print_preview' ? 'bg-slate-700 text-white' : 'text-slate-400 hover:bg-slate-800'}`}>
        <Printer size={18} /> <span className="font-semibold">列印整批匯款明細</span>
      </button>
      <div className="px-4 py-2 text-[10px] font-bold text-slate-500 uppercase tracking-widest mt-4 border-t border-slate-800 pt-4">資料庫 (Table)</div>
      <button onClick={() => setViewMode('table_base')} className={`mx-2 flex items-center gap-3 px-4 py-2 text-xs rounded-xl transition-all ${viewMode === 'table_base' ? 'bg-slate-800 text-white' : 'text-slate-500 hover:bg-slate-800'}`}>R_base (匯款人)</button>
      <button onClick={() => setViewMode('table_cust')} className={`mx-2 flex items-center gap-3 px-4 py-2 text-xs rounded-xl transition-all ${viewMode === 'table_cust' ? 'bg-slate-800 text-white' : 'text-slate-500 hover:bg-slate-800'}`}>R_CUST (收款人)</button>
      <button onClick={() => setViewMode('table_day')} className={`mx-2 flex items-center gap-3 px-4 py-2 text-xs rounded-xl transition-all ${viewMode === 'table_day' ? 'bg-slate-800 text-white' : 'text-slate-500 hover:bg-slate-800'}`}>R_DAY (批號)</button>
      <button onClick={() => setViewMode('table_detl')} className={`mx-2 flex items-center gap-3 px-4 py-2 text-xs rounded-xl transition-all ${viewMode === 'table_detl' ? 'bg-slate-800 text-white' : 'text-slate-500 hover:bg-slate-800'}`}>R_DETL (明細)</button>
    </div>
  );

  return (
    <div className="flex h-screen bg-slate-100 font-sans text-slate-800 overflow-hidden text-[14px]">
      {/* 側邊欄 */}
      <aside className="hidden md:flex w-64 bg-[#0f172a] text-white flex-col shadow-2xl shrink-0 z-30">
        <div className="p-6 flex items-center gap-3 border-b border-slate-800 bg-slate-900/50">
          <div className="w-8 h-8 bg-blue-500 rounded-lg flex items-center justify-center shadow-lg"><Database size={18} /></div>
          <h1 className="text-base font-black tracking-tighter uppercase">RemitPro <span className="text-blue-400">DB</span></h1>
        </div>
        <div className="flex-1 overflow-y-auto py-2"><NavLinks /></div>
      </aside>

      {/* 主內容區 */}
      <main className="flex-1 flex flex-col overflow-hidden">
        <header className="h-16 bg-white border-b flex items-center px-6 shrink-0 z-20 shadow-sm">
          <div className="flex items-center gap-2 text-slate-400 text-sm">
            <span>系統</span> <ChevronRight size={14} />
            <h2 className="font-bold text-slate-800 uppercase">
                {viewMode.includes('form_remitter') && "匯款人資料建檔"}
                {viewMode.includes('form_recipient') && "收款人資料建檔"}
                {viewMode.includes('form_day') && "匯款批號輸入"}
                {viewMode.includes('form_detl') && "匯款明細輸入"}
                {viewMode.includes('print_preview') && "報表列印預覽"}
                {viewMode.startsWith('table') && `資料表檢視: ${viewMode.split('_')[1].toUpperCase()}`}
            </h2>
          </div>
        </header>

        <div className="flex-1 overflow-y-auto p-6 bg-slate-50">
          <div className="max-w-5xl mx-auto">
            
            {/* 匯款人資料建檔 */}
            {viewMode === 'form_remitter' && (
              <form onSubmit={(e) => {e.preventDefault(); setDataListBase([{...formRemitter, id: 1}]); handleShowSuccess();}} className="bg-white p-8 rounded-3xl shadow-sm border space-y-6">
                <h3 className="font-bold text-lg text-blue-600 border-b pb-4">匯款人資料建檔</h3>
                <div className="grid grid-cols-2 gap-6">
                  <div className="space-y-1"><label className="text-xs font-bold text-slate-400">行號 (BANK_ID)</label><input value={formRemitter.bankId} onChange={e => setFormRemitter({...formRemitter, bankId: e.target.value})} className="w-full bg-slate-50 border rounded-xl px-4 py-3 font-mono font-bold" /></div>
                  <div className="space-y-1"><label className="text-xs font-bold text-slate-400">名稱 (NAME)</label><input value={formRemitter.name} onChange={e => setFormRemitter({...formRemitter, name: e.target.value})} className="w-full bg-slate-50 border rounded-xl px-4 py-3 font-bold" /></div>
                  <div className="col-span-2 space-y-1"><label className="text-xs font-bold text-slate-400">匯款種類 (TYPE)</label>
                    <div className="flex gap-4">
                      {['10','11'].map(t => <button key={t} type="button" onClick={() => setFormRemitter({...formRemitter, type: t})} className={`flex-1 py-3 rounded-xl border-2 font-bold ${formRemitter.type === t ? 'bg-blue-600 border-blue-600 text-white' : 'bg-white text-slate-400'}`}>{t === '10' ? '10 - 一般匯款' : '11 - 公庫匯款'}</button>)}
                    </div>
                  </div>
                  <div className="col-span-2 space-y-3 pt-4 border-t">
                    <input value={formRemitter.note1} onChange={e => setFormRemitter({...formRemitter, note1: e.target.value})} className="w-full bg-slate-50 border rounded-xl px-4 py-2 text-xs" placeholder="Note 1" />
                    <input value={formRemitter.note2} onChange={e => setFormRemitter({...formRemitter, note2: e.target.value})} className="w-full bg-slate-50 border rounded-xl px-4 py-2 text-xs" placeholder="Note 2" />
                    <input value={formRemitter.note3} onChange={e => setFormRemitter({...formRemitter, note3: e.target.value})} className="w-full bg-slate-50 border rounded-xl px-4 py-2 text-xs" placeholder="Note 3" />
                  </div>
                </div>
                <div className="flex justify-end"><button type="submit" className="px-10 py-3 bg-blue-600 text-white font-bold rounded-xl shadow-lg">儲存</button></div>
              </form>
            )}

            {/* 收款人資料建檔 */}
            {viewMode === 'form_recipient' && (
              <form onSubmit={(e) => {e.preventDefault(); setDataListCust(prev => [{...formRecipient, id: Date.now()}, ...prev]); setFormRecipient({custCode:'', custName:'', custAccount:'', bankAccount:''}); handleShowSuccess();}} className="bg-white p-8 rounded-3xl shadow-sm border space-y-6">
                <h3 className="font-bold text-lg text-indigo-600 border-b pb-4">收款人資料建檔</h3>
                <div className="grid grid-cols-2 gap-6">
                  <div className="space-y-1"><label className="text-xs font-bold text-slate-400">代碼 (CUST_CODE)</label><input value={formRecipient.custCode} onChange={e => setFormRecipient({...formRecipient, custCode: e.target.value})} className="w-full bg-slate-50 border rounded-xl px-4 py-3 font-mono" /></div>
                  <div className="space-y-1"><label className="text-xs font-bold text-slate-400">姓名 (CUST_NAME)</label><input value={formRecipient.custName} onChange={e => setFormRecipient({...formRecipient, custName: e.target.value})} className="w-full bg-slate-50 border rounded-xl px-4 py-3 font-bold" /></div>
                  <div className="space-y-1"><label className="text-xs font-bold text-slate-400">帳號 (CUST_ACCOUNT)</label><input value={formRecipient.custAccount} onChange={e => setFormRecipient({...formRecipient, custAccount: e.target.value})} className="w-full bg-slate-50 border rounded-xl px-4 py-3 font-mono" /></div>
                  <div className="space-y-1"><label className="text-xs font-bold text-slate-400">解款行號 (BANK_ACCOUNT)</label><input value={formRecipient.bankAccount} onChange={e => setFormRecipient({...formRecipient, bankAccount: e.target.value})} className="w-full bg-slate-50 border rounded-xl px-4 py-3 font-mono text-indigo-600 font-bold" /></div>
                </div>
                <div className="flex justify-end"><button type="submit" className="px-10 py-3 bg-indigo-600 text-white font-bold rounded-xl shadow-lg">新增資料</button></div>
              </form>
            )}

            {/* 匯款批號輸入 */}
            {viewMode === 'form_day' && (
              <form onSubmit={(e) => {e.preventDefault(); setDataListDay(prev => [{...formDay, id: Date.now()}, ...prev]); handleShowSuccess();}} className="bg-white p-8 rounded-3xl shadow-sm border space-y-6">
                <h3 className="font-bold text-lg text-amber-600 border-b pb-4">匯款批號輸入</h3>
                <div className="grid grid-cols-2 gap-6">
                  <div className="space-y-1"><label className="text-xs font-bold text-slate-400">匯款日期</label><input value={formDay.remitDate} onChange={e => setFormDay({...formDay, remitDate: e.target.value})} className="w-full bg-slate-50 border rounded-xl px-4 py-3 font-mono font-bold" /></div>
                  <div className="space-y-1"><label className="text-xs font-bold text-slate-400">批號編碼</label><input value={formDay.batchNo} onChange={e => setFormDay({...formDay, batchNo: e.target.value})} className="w-full bg-slate-50 border rounded-xl px-4 py-3 font-mono font-bold" /></div>
                  <div className="col-span-2 space-y-1">
                    <label className="text-xs font-bold text-slate-400">用途描述</label>
                    <textarea value={formDay.purpose} onChange={e => setFormDay({...formDay, purpose: e.target.value})} className="w-full bg-slate-50 border rounded-xl px-4 py-3 h-24 font-bold" />
                  </div>
                </div>
                <div className="flex justify-end"><button type="submit" className="px-10 py-3 bg-amber-600 text-white font-bold rounded-xl shadow-lg">更新</button></div>
              </form>
            )}

            {/* 匯款明細輸入 */}
            {viewMode === 'form_detl' && (
              <div className="space-y-6">
                <div className="bg-white p-8 rounded-3xl shadow-sm border space-y-6">
                  <h3 className="font-bold text-lg text-emerald-600 border-b pb-4">匯款明細輸入</h3>
                  <div className="relative">
                    <label className="text-xs font-bold text-slate-400 block mb-1">搜尋收款人 (依姓名或代碼)</label>
                    <input type="text" className="w-full px-4 py-3 bg-slate-50 border-2 rounded-xl font-bold" value={custSearchTerm} onChange={e => {setCustSearchTerm(e.target.value); setShowCustSuggestions(true);}} />
                    {showCustSuggestions && getSuggestions().length > 0 && (
                      <div className="absolute top-full left-0 right-0 mt-1 bg-white border rounded-xl shadow-xl z-50 divide-y">
                         {getSuggestions().map(c => <button key={c.id} type="button" onClick={() => selectSuggestion(c)} className="w-full px-4 py-3 hover:bg-emerald-50 text-left flex justify-between"><span>{c.custName} <small>({c.custCode})</small></span> <span className="text-xs text-slate-400">{c.bankAccount}</span></button>)}
                      </div>
                    )}
                  </div>
                  <div className="grid grid-cols-4 gap-4">
                    {[['代碼', formDetail.custCode], ['姓名', formDetail.custName], ['帳號', formDetail.custAccount], ['解款行', formDetail.bankAccount]].map(([l, v]) => (
                      <div key={l} className="p-3 bg-slate-50 rounded-xl border text-center"><div className="text-[10px] text-slate-400 font-bold">{l}</div><div className="font-bold truncate">{v || '---'}</div></div>
                    ))}
                  </div>
                  <div className="grid grid-cols-3 gap-6 p-6 bg-emerald-50/30 rounded-2xl border">
                     <div className="space-y-1"><label className="text-xs font-bold">匯款金額</label><input type="number" className="w-full px-4 py-3 rounded-xl border-2 font-black text-xl text-emerald-700" value={formDetail.amount} onChange={e => setFormDetail({...formDetail, amount: e.target.value})} /></div>
                     <div className="space-y-1"><label className="text-xs font-bold">手續費</label><input type="number" className="w-full px-4 py-3 rounded-xl border-2 font-bold text-xl text-rose-500" value={formDetail.fee} onChange={e => setFormDetail({...formDetail, fee: e.target.value})} /></div>
                     <div className="space-y-1"><label className="text-xs font-bold">備註</label><select className="w-full px-4 py-3 rounded-xl border-2 font-bold bg-white" value={formDetail.memo} onChange={e => setFormDetail({...formDetail, memo: e.target.value})}>{NOTE_OPTIONS.map(o => <option key={o} value={o}>{o}</option>)}</select></div>
                  </div>
                  <div className="flex justify-end"><button onClick={() => {setTempDetailList(prev => [...prev, {...formDetail, id: Date.now()}]); setFormDetail({...formDetail, amount:'', fee:'0', custCode:'', custName:'', custAccount:'', bankAccount:'', memo:'薪資'}); setCustSearchTerm('');}} className="px-10 py-3 bg-emerald-600 text-white font-bold rounded-xl shadow-lg">加入暫存清單</button></div>
                </div>

                {tempDetailList.length > 0 && (
                  <div className="bg-white rounded-3xl border shadow-sm overflow-hidden">
                    <div className="px-8 py-4 bg-slate-50 border-b flex justify-between items-center">
                      <h4 className="text-xs font-bold text-slate-500">暫存清單 ({tempDetailList.length})</h4>
                      <button onClick={() => {setDataListDetl(prev => [...tempDetailList, ...prev]); setTempDetailList([]); handleShowSuccess();}} className="px-6 py-2 bg-slate-900 text-white rounded-xl text-xs font-bold">批次寫入 R_DETL</button>
                    </div>
                    <table className="w-full text-xs">
                      <tbody className="divide-y">
                        {tempDetailList.map(item => (
                          <tr key={item.id} className="hover:bg-slate-50">
                            <td className="py-3 px-6 font-bold">{item.custName}</td>
                            <td className="px-6 font-mono text-slate-500">{item.custAccount}</td>
                            <td className="px-6 text-right font-black text-emerald-600">${parseFloat(item.amount).toLocaleString()}</td>
                            <td className="px-6 text-center"><button onClick={() => deleteRow(item.id, 'temp')} className="text-slate-300 hover:text-rose-500"><Trash2 size={16}/></button></td>
                          </tr>
                        ))}
                      </tbody>
                    </table>
                  </div>
                )}
              </div>
            )}

            {/* 資料表檢視 */}
            {viewMode.startsWith('table') && (
              <div className="bg-white rounded-2xl shadow-sm border overflow-hidden flex flex-col h-[calc(100vh-14rem)]">
                <div className="px-6 py-4 border-b bg-slate-50 text-xs font-bold text-slate-500">資料表: R_{viewMode.split('_')[1].toUpperCase()}</div>
                <div className="flex-1 overflow-auto">
                  <table className="w-full text-xs">
                    <thead className="bg-slate-50 border-b sticky top-0 z-10 text-slate-400 font-bold">
                      <tr>
                        <th className="py-3 px-4 w-12 text-center">#</th>
                        {viewMode === 'table_base' && <><th className="px-4 text-left">行號 (BANK_ID)</th><th className="px-4 text-left">名稱 (NAME)</th><th className="px-4 text-left">種類 (TYPE)</th><th className="px-4 text-left">Note 1</th><th className="px-4 text-left">Note 2</th><th className="px-4 text-left">Note 3</th></>}
                        {viewMode === 'table_cust' && <><th className="px-4 text-left">代碼 (CUST_CODE)</th><th className="px-4 text-left">姓名 (CUST_NAME)</th><th className="px-4 text-left">帳號 (CUST_ACCOUNT)</th><th className="px-4 text-left">解款行 (BANK_ACCOUNT)</th></>}
                        {viewMode === 'table_day' && <><th className="px-4 text-left">匯款日期</th><th className="px-4 text-left">批號編碼</th><th className="px-4 text-left">用途描述</th></>}
                        {viewMode === 'table_detl' && <><th className="px-4 text-left">代碼 (CUST_CODE)</th><th className="px-4 text-left">姓名 (CUST_NAME)</th><th className="px-4 text-left">帳號 (CUST_ACCOUNT)</th><th className="px-4 text-left">解款行 (BANK_ACCOUNT)</th><th className="px-4 text-right">匯款金額</th><th className="px-4 text-right">手續費</th><th className="px-4 text-center">備註 (MEMO)</th></>}
                        <th className="w-16 px-4 text-center">刪除</th>
                      </tr>
                    </thead>
                    <tbody className="divide-y">
                      {(viewMode === 'table_base' ? dataListBase : viewMode === 'table_cust' ? dataListCust : viewMode === 'table_day' ? dataListDay : dataListDetl).map((item, idx) => (
                        <tr key={item.id} className="hover:bg-blue-50/30">
                          <td className="py-3 text-center text-slate-400 font-mono">{idx + 1}</td>
                          {viewMode === 'table_base' && <><td className="px-4 font-mono font-bold text-blue-600">{item.bankId}</td><td className="px-4 font-bold">{item.name}</td><td className="px-4">{item.type}</td><td className="px-4 truncate max-w-[120px]">{item.note1}</td><td className="px-4 truncate max-w-[100px]">{item.note2}</td><td className="px-4 truncate max-w-[100px]">{item.note3}</td></>}
                          {viewMode === 'table_cust' && <><td className="px-4 font-mono font-bold">{item.custCode}</td><td className="px-4 font-bold">{item.custName}</td><td className="px-4 font-mono">{item.custAccount}</td><td className="px-4 font-mono text-indigo-500 font-bold">{item.bankAccount}</td></>}
                          {viewMode === 'table_day' && <><td className="px-4 font-mono font-bold">{item.remitDate}</td><td className="px-4 font-mono font-bold">{item.batchNo}</td><td className="px-4">{item.purpose}</td></>}
                          {viewMode === 'table_detl' && <><td className="px-4 font-mono">{item.custCode}</td><td className="px-4 font-bold">{item.custName}</td><td className="px-4 font-mono">{item.custAccount}</td><td className="px-4 font-mono text-indigo-500 font-bold">{item.bankAccount}</td><td className="px-4 text-right font-bold text-emerald-600">{parseFloat(item.amount).toLocaleString()}</td><td className="px-4 text-right text-rose-500">{item.fee}</td><td className="px-4 text-center">{item.memo}</td></>}
                          <td className="px-4 text-center"><button onClick={() => deleteRow(item.id, viewMode.split('_')[1])} className="text-slate-200 hover:text-rose-500"><Trash2 size={14} /></button></td>
                        </tr>
                      ))}
                    </tbody>
                  </table>
                </div>
              </div>
            )}

            {/* 列印預覽區 */}
            {viewMode === 'print_preview' && (
              <div className="space-y-6">
                <div className="flex justify-between items-center bg-white p-4 rounded-2xl shadow-sm border">
                  <h3 className="font-bold text-slate-800">報表列印預覽</h3>
                  <button onClick={handlePrint} className="bg-blue-600 text-white px-8 py-3 rounded-xl font-bold shadow-lg flex items-center gap-2 hover:bg-blue-700 transition-all"><Printer size={18}/> 列印</button>
                </div>
                <div id="print-area" className="bg-white shadow-2xl p-12 mx-auto w-[277mm] overflow-auto">
                   <div className="flex justify-between items-end mb-6">
                      <h1 className="text-2xl font-bold tracking-[0.4em]">整 批 匯 款 資 料 清 單</h1>
                      <div className="text-sm">頁次：1</div>
                   </div>
                   <div className="grid grid-cols-2 gap-4 mb-4 text-[13px] font-mono">
                      <div>
                        <p>匯款人：{dataListBase[0]?.name}</p>
                        <p>匯款行：{dataListBase[0]?.bankId} [台灣銀行斗六分行]</p>
                        <p>匯款日：{dataListDay[0]?.remitDate}</p>
                      </div>
                      <div className="text-right">
                        <p>製表日：{getTodayYYMMDDD()}</p>
                        <p>用途：{dataListDay[0]?.purpose}</p>
                        <p>批號：{dataListDay[0]?.batchNo}</p>
                      </div>
                   </div>
                   <table className="w-full text-left font-mono text-[13px]">
                     <thead>
                       <tr className="border-b border-black border-dashed font-bold">
                         <th className="py-2">序號</th>
                         <th>解款行</th>
                         <th>收款人帳號</th>
                         <th className="text-right">匯款金額</th>
                         <th className="text-right">手續費</th>
                         <th className="px-4">收款人姓名</th>
                         <th>備註</th>
                       </tr>
                     </thead>
                     <tbody>
                       {dataListDetl.map((item, idx) => (
                         <tr key={item.id} className="border-b border-gray-100">
                           <td className="py-1.5">{idx + 1}</td>
                           <td>{item.bankAccount}</td>
                           <td>{item.custAccount}</td>
                           <td className="text-right">{parseFloat(item.amount).toLocaleString()}</td>
                           <td className="text-right">{parseFloat(item.fee).toLocaleString()}</td>
                           <td className="px-4">{item.custName}</td>
                           <td>{item.memo}</td>
                         </tr>
                       ))}
                     </tbody>
                   </table>
                   <div className="mt-8 flex justify-between font-bold">
                     <span>{dataListBase[0]?.note1}</span>
                   </div>
                   <div className="mt-4 text-[11px] text-gray-500">
                     <p>{dataListBase[0]?.note3}</p>
                   </div>
                </div>
              </div>
            )}

          </div>
        </div>
      </main>

      {/* 通知元件 */}
      {showSuccess && (
        <div className="fixed top-20 right-8 z-50 bg-emerald-500 text-white px-6 py-3 rounded-xl shadow-lg flex items-center gap-3 font-bold animate-bounce">
          <CheckCircle2 size={18} /> 執行成功
        </div>
      )}
    </div>
  );
};

export default App;
