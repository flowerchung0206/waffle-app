<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>華夫格格管理系統</title>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Noto+Sans+TC', sans-serif; background-color: #FDF8F5; margin: 0; padding: 0; }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        input[type="number"]::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }
        .receipt-container { background-color: #FDF8F5; padding: 1.5rem; border-radius: 2.5rem; box-shadow: inset 0 2px 10px rgba(0,0,0,0.05); }
        .animate-fade { animation: fadeIn 0.2s ease-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useMemo, useEffect, memo, useCallback } = React;

        // --- 安全圖示組件 (SVG) ---
        const Icon = ({ name, size = 20, className = "" }) => {
            const icons = {
                plus: <path d="M12 5v14M5 12h14" />,
                minus: <path d="M5 12h14" />,
                user: <g><path d="M19 21v-2a4 4 0 0 0-4-4H9a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></g>,
                trash: <g><path d="M3 6h18"/><path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6"/><path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"/></g>,
                edit: <g><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"/></g>,
                cart: <g><circle cx="8" cy="21" r="1"/><circle cx="19" cy="21" r="1"/><path d="M2.05 2.05h2l2.66 12.42a2 2 0 0 0 2 1.58h9.78a2 2 0 0 0 1.95-1.57l1.65-7.43H5.12"/></g>,
                chef: <g><path d="M6 18h12"/><path d="M12 10V4.5a2.5 2.5 0 0 0-5 0V10a2 2 0 0 1-2 2h0a2 2 0 0 1-2-2V8"/><path d="M12 10V4.5a2.5 2.5 0 0 1 5 0V10a2 2 0 0 0 2 2h0a2 2 0 0 0 2-2V8"/><path d="M18 18a4 4 0 1 1-8 0"/></g>,
                back: <g><polyline points="15 18 9 12 15 6"/></g>,
                target: <g><circle cx="12" cy="12" r="10"/><circle cx="12" cy="12" r="6"/><circle cx="12" cy="12" r="2"/></g>,
                pin: <g><path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z"/><circle cx="12" cy="10" r="3"/></g>,
                list: <g><line x1="8" x2="21" y1="6" y2="6"/><line x1="8" x2="21" y1="12" y2="12"/><line x1="8" x2="21" y1="18" y2="18"/><line x1="3" x2="3.01" y1="6" y2="6"/><line x1="3" x2="3.01" y1="12" y2="12"/><line x1="3" x2="3.01" y1="18" y2="18"/></g>,
                truck: <g><rect width="16" height="10" x="2" y="6" rx="2"/><path d="M18 8h3l3 3v5h-6"/><circle cx="6.5" cy="18.5" r="2.5"/><circle cx="18.5" cy="18.5" r="2.5"/></g>,
                card: <g><rect width="20" height="14" x="2" y="5" rx="2"/><line x1="2" x2="22" y1="10" y2="10"/></g>,
                send: <g><line x1="22" y1="2" x2="11" y2="13"/><polyline points="22 2 15 22 11 13 2 9 22 2"/></g>,
                check: <polyline points="20 6 9 17 4 12"/>,
                gift: <g><rect width="18" height="14" x="3" y="8" rx="2"/><path d="M12 5a3 3 0 1 0-5.997.125 4 4 0 0 0 5.997 0A4 4 0 0 0 17.997 5.125 3 3 0 1 0 12 5Z"/><line x1="12" x2="12" y1="22" y2="8"/></g>
            };
            return (
                <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2.5" strokeLinecap="round" strokeLinejoin="round" className={className}>
                    {icons[name] || <circle cx="12" cy="12" r="10" />}
                </svg>
            );
        };

        const flavorNames = { original: '原味', strawberry: '草莓', cocoa: '可可', matcha: '抹茶' };
        const extraNames = { cream: '鮮奶油', bag: '加購提袋', receipt: '手開收據', sugar: '加糖粉', fork: '加叉子' };
        const bankInfo = { bankName: "中國信託 (822)", accountName: "彭永翔", accountNumber: "1755 4046 3472" };

        const productCategories = {
            '8入': [
                { id: '8_orig', label: '8入 原味', price: 75, balls: 8, flavors: ['original'] },
                { id: '8_straw', label: '8入 草莓', price: 95, balls: 8, flavors: ['strawberry'] },
                { id: '8_matcha', label: '8入 抹茶', price: 80, balls: 8, flavors: ['matcha'] },
                { id: '8_cocoa', label: '8入 可可', price: 75, balls: 8, flavors: ['cocoa'] },
            ],
            '4入': [
                { id: '4_orig', label: '4入 原味', price: 45, balls: 4, flavors: ['original'] },
                { id: '4_straw', label: '4入 草莓', price: 60, balls: 4, flavors: ['strawberry'] },
                { id: '4_matcha', label: '4入 抹茶', price: 55, balls: 4, flavors: ['matcha'] },
                { id: '4_cocoa', label: '4入 可可', price: 50, balls: 4, flavors: ['cocoa'] },
            ],
            '4入雙色': [
                { id: '4_os', label: '原味+草莓', price: 60, balls: 4, flavors: ['original', 'strawberry'] },
                { id: '4_om', label: '原味+抹茶', price: 55, balls: 4, flavors: ['original', 'matcha'] },
                { id: '4_oc', label: '原味+可可', price: 50, balls: 4, flavors: ['original', 'cocoa'] },
                { id: '4_sm', label: '草莓+抹茶', price: 65, balls: 4, flavors: ['strawberry', 'matcha'] },
                { id: '4_sc', label: '草莓+可可', price: 60, balls: 4, flavors: ['strawberry', 'cocoa'] },
                { id: '4_cm', label: '抹茶+可可', price: 60, balls: 4, flavors: ['matcha', 'cocoa'] },
            ],
            '綜合': [
                { id: '8_mix', label: '8入 四色綜合', price: 110, balls: 8, flavors: ['original', 'strawberry', 'cocoa', 'matcha'] },
                { id: '4_mix', label: '4入 四色綜合', price: 65, balls: 4, flavors: ['original', 'strawberry', 'cocoa', 'matcha'] },
            ],
            '真空包': [
                { id: 'v_40_o', label: '真空40 原味', price: 320, balls: 40, flavors: ['original'], vType: 40 },
                { id: 'v_40_s', label: '真空40 草莓', price: 500, balls: 40, flavors: ['strawberry'], vType: 40 },
                { id: 'v_40_c', label: '真空40 可可', price: 385, balls: 40, flavors: ['cocoa'], vType: 40 },
                { id: 'v_40_m', label: '真空40 抹茶', price: 380, balls: 40, flavors: ['matcha'], vType: 40 },
                { id: 'v_80_o', label: '真空80 原味', price: 550, balls: 80, flavors: ['original'], vType: 80 },
                { id: 'v_80_s', label: '真空80 草莓', price: 790, balls: 80, flavors: ['strawberry'], vType: 80 },
                { id: 'v_80_c', label: '真空80 可可', price: 700, balls: 80, flavors: ['cocoa'], vType: 80 },
                { id: 'v_80_m', label: '真空80 抹茶', price: 750, balls: 80, flavors: ['matcha'], vType: 80 },
            ]
        };

        const calculatePacking = (items) => {
            let large = 0, small = 0, v40 = 0, v80 = 0;
            (items || []).forEach(i => {
                const id = i.id || '';
                if (id.includes('8_') || id === '8_mix') large += (i.qty || 0);
                else if (id.includes('4_') || id === '4_mix') small += (i.qty || 0);
                if (i.vType === 40) v40 += (i.qty || 0);
                if (i.vType === 80) v80 += (i.qty || 0);
            });
            let bag6 = 0, bag4 = 0;
            let tL = large, tS = small;
            while (tL >= 9 || tS >= 12) { bag6++; tL = Math.max(0, tL - 9); tS = Math.max(0, tS - 12); }
            while (tL > 0 || tS > 0) { bag4++; tL = Math.max(0, tL - 5); tS = Math.max(0, tS - 6); }
            return { bag6, bag4, cooler: Math.ceil(v40 / 2) + v80 };
        };

        const getDoughDemand = (items) => {
            const counts = { original: 0, strawberry: 0, cocoa: 0, matcha: 0 };
            (items || []).forEach(i => {
                const totalBalls = (i.balls || 0) * (i.qty || 0);
                const flavors = i.flavors || [];
                flavors.forEach(f => { if (counts[f] !== undefined) counts[f] += totalBalls / flavors.length; });
            });
            return Object.entries(counts)
                .filter(([_, v]) => v > 0)
                .map(([k, v]) => ({
                    name: flavorNames[k],
                    batches: Math.floor(v / 80),
                    remainder: Math.round(v % 80),
                    total: Math.round(v)
                }));
        };

        // --- 主應用邏輯 ---
        const App = () => {
            const [view, setView] = useState('main'); // 'main' or 'receipt'
            const [activeTab, setActiveTab] = useState('order');
            const [orders, setOrders] = useState([]);
            const [editingId, setEditingId] = useState(null); // 修改中的訂單 ID
            const [orderCategory, setOrderCategory] = useState('8入');
            const [currentOrder, setCurrentOrder] = useState({
                name: '', phone: '', address: '', taxId: '', schoolCompany: '', deliveryTime: '', 
                deliveryMethod: '自取', paymentMethod: '店內結帳', items: [], 
                extras: { cream: false, bag: true, receipt: false, sugar: false, fork: false }, note: ''
            });
            const [selectedReceipt, setSelectedReceipt] = useState(null);

            useEffect(() => {
                const saved = localStorage.getItem('gege_vFinal_Complete_v2');
                if (saved) setOrders(JSON.parse(saved));
            }, []);

            useEffect(() => {
                localStorage.setItem('gege_vFinal_Complete_v2', JSON.stringify(orders));
            }, [orders]);

            const doughTotal = useMemo(() => {
                const totals = { original: 0, strawberry: 0, cocoa: 0, matcha: 0 };
                orders.forEach(o => {
                    (o.items || []).forEach(i => {
                        const balls = (i.balls || 0) * (i.qty || 0);
                        const flavors = i.flavors || [];
                        flavors.forEach(f => { if (totals[f] !== undefined) totals[f] += balls / flavors.length; });
                    });
                });
                return Object.entries(totals).map(([k, v]) => ({
                    name: flavorNames[k], batches: Math.floor(v / 80), remainder: Math.round(v % 80), total: Math.round(v)
                })).filter(d => d.total > 0);
            }, [orders]);

            const handleAddItem = (opt) => {
                const existing = currentOrder.items.find(i => i.id === opt.id);
                if (existing) {
                    setCurrentOrder({
                        ...currentOrder,
                        items: currentOrder.items.map(i => i.id === opt.id ? { ...i, qty: (i.qty || 0) + 1 } : i)
                    });
                } else {
                    setCurrentOrder({
                        ...currentOrder,
                        items: [...currentOrder.items, { ...opt, qty: 1 }]
                    });
                }
            };

            const handleSaveOrder = () => {
                if (!currentOrder.name || currentOrder.items.length === 0) return;
                const p = calculatePacking(currentOrder.items);
                const iSum = currentOrder.items.reduce((s, i) => s + (i.price * (i.qty || 0)), 0);
                const bSum = currentOrder.extras.bag ? (p.bag6 + p.bag4) * 2 : 0;
                const totalAmount = iSum + bSum;

                if (editingId) {
                    // 修改模式
                    setOrders(orders.map(o => o.id === editingId ? { ...currentOrder, id: editingId, totalAmount, timestamp: new Date().toLocaleString() } : o));
                    setEditingId(null);
                } else {
                    // 新增模式
                    const newOrder = { ...currentOrder, id: Date.now(), totalAmount, timestamp: new Date().toLocaleString() };
                    setOrders([newOrder, ...orders]);
                }

                // 重置
                setCurrentOrder({ name: '', phone: '', address: '', taxId: '', schoolCompany: '', deliveryTime: '', deliveryMethod: '自取', paymentMethod: '店內結帳', items: [], extras: { cream: false, bag: true, receipt: false, sugar: false, fork: false }, note: '' });
                setActiveTab('list');
            };

            const handleEditClick = (order) => {
                setCurrentOrder(order);
                setEditingId(order.id);
                setActiveTab('order');
            };

            const onCopyLine = (order) => {
                const p = calculatePacking(order.items);
                let payInfo = `【付款方式：${order.paymentMethod}】`;
                if (order.paymentMethod === '匯款') {
                    payInfo += `\n🏦 中信(822) ${bankInfo.accountNumber}\n👤 戶名：${bankInfo.accountName}\n(匯款後請提供末五碼核對)`;
                }

                let itemsText = order.items.map(i => `• ${i.label} x ${i.qty} = $${i.price * i.qty}`).join('\n');
                let extraList = [];
                if (order.extras.cream) extraList.push('鮮奶油');
                if (order.extras.sugar) extraList.push('加糖粉');
                if (order.extras.fork) extraList.push('加叉子');
                if (order.extras.receipt) extraList.push('附收據');
                let extrasText = extraList.length > 0 ? `\n• 其他要求：${extraList.join('、')}` : '';

                let bagText = order.extras.bag ? `\n• 提袋費($2)：${p.bag6 + p.bag4} 個 = $${(p.bag6 + p.bag4) * 2}` : '';
                let coolerText = p.cooler > 0 ? `\n🎁 真空包贈送保冷袋：${p.cooler} 個` : '';

                const fullText = `【華夫格格 - 訂單對帳單】\n客戶：${order.name}\n配送：${order.deliveryMethod}\n時間：${order.deliveryTime?.replace('T',' ')}\n${payInfo}\n--------------------\n${itemsText}${extrasText}${bagText}${coolerText}\n--------------------\n💰 總計金額：$ ${order.totalAmount}\n備註：${order.note || '無'}\n\n感謝您的訂購！`;
                
                const textArea = document.createElement("textarea");
                textArea.value = fullText; document.body.appendChild(textArea);
                textArea.select(); document.execCommand('copy'); document.body.removeChild(textArea);
                alert("對帳文字已複製成功！");
                setView('main');
            };

            // --- 視圖：收據明細頁面 ---
            if (view === 'receipt' && selectedReceipt) {
                const p = calculatePacking(selectedReceipt.items);
                return (
                    <div className="min-h-screen bg-white animate-fade flex flex-col text-left">
                        <div className="p-4 border-b flex items-center bg-white sticky top-0 z-50">
                            <button onClick={() => setView('main')} className="p-2 text-[#8E7D6F] flex items-center font-bold">
                                <Icon name="back" size={24} /> 返回
                            </button>
                            <h2 className="flex-1 text-center font-black text-[#5E503F]">對帳明細</h2>
                            <div className="w-10"></div>
                        </div>
                        <div className="flex-1 overflow-y-auto p-4 pb-40">
                            <div className="receipt-container">
                                <div className="text-center mb-6 border-b-2 border-dashed border-[#D5BDAF]/30 pb-4">
                                    <h3 className="text-2xl font-black text-[#D5BDAF] tracking-widest">華夫格格</h3>
                                    <p className="text-[10px] opacity-40 uppercase tracking-widest">Belgian Waffles</p>
                                </div>
                                <div className="text-center space-y-2 mb-6 text-left flex flex-col items-center">
                                    <p className="text-4xl font-black text-[#5E503F] tracking-tighter">{selectedReceipt.name}</p>
                                    <div className="flex justify-center gap-2">
                                        <span className="bg-[#8E7D6F] text-white px-3 py-1 rounded-full text-[9px] font-black uppercase"><Icon name="truck" size={10} className="inline mr-1"/>{selectedReceipt.deliveryMethod}</span>
                                        <span className="bg-[#D5BDAF] text-white px-3 py-1 rounded-full text-[9px] font-black uppercase"><Icon name="card" size={10} className="inline mr-1"/>{selectedReceipt.paymentMethod}</span>
                                    </div>
                                    <p className="text-[11px] font-bold text-gray-400">{selectedReceipt.phone || '無電話紀錄'}</p>
                                    {selectedReceipt.address && <p className="text-[11px] font-bold text-[#8E7D6F] flex items-center justify-center gap-1"><Icon name="pin" size={10}/>{selectedReceipt.address}</p>}
                                </div>
                                
                                <div className="space-y-4 mb-6 font-bold border-t border-gray-100 pt-4">
                                    <p className="text-[10px] text-[#D5BDAF] uppercase tracking-widest text-left">● 訂購品項</p>
                                    {selectedReceipt.items.map((i, idx) => (
                                        <div key={idx} className="flex justify-between items-end border-b border-gray-50 pb-2">
                                            <div className="text-left leading-tight"><p className="text-lg">• {i.label}</p><p className="text-[10px] opacity-40">$ {i.price} x {i.qty}</p></div>
                                            <span className="text-[#8E7D6F]">$ {i.price * i.qty}</span>
                                        </div>
                                    ))}
                                    
                                    <p className="text-[10px] text-[#D5BDAF] uppercase tracking-widest pt-2 text-left">● 加購與服務</p>
                                    <div className="space-y-1 text-sm pt-1 text-left">
                                        {Object.entries(selectedReceipt.extras).map(([key, value]) => {
                                            if (key === 'bag') return null;
                                            return value ? (
                                                <div key={key} className="flex justify-between text-[#5E503F] border-b border-gray-50 pb-1">
                                                    <span>• {extraNames[key]}</span>
                                                    <span className="text-[#8E7D6F]">$ 0</span>
                                                </div>
                                            ) : null;
                                        })}
                                        {selectedReceipt.extras.bag && (p.bag6 > 0 || p.bag4 > 0) && (
                                            <React.Fragment>
                                                {p.bag6 > 0 && <div className="flex justify-between"><span>• 專用 6杯提袋 ($2) x {p.bag6}</span><span>$ {p.bag6 * 2}</span></div>}
                                                {p.bag4 > 0 && <div className="flex justify-between"><span>• 專用 4杯提袋 ($2) x {p.bag4}</span><span>$ {p.bag4 * 2}</span></div>}
                                            </React.Fragment>
                                        )}
                                        {p.cooler > 0 && (
                                            <div className="flex justify-between text-blue-600 font-black pt-1">
                                                <span>• 贈送：真空包專用保冷袋</span>
                                                <span>x {p.cooler} 個 ($0)</span>
                                            </div>
                                        )}
                                    </div>
                                </div>

                                <div className="bg-[#8E7D6F] p-6 rounded-[2rem] flex justify-between items-center text-white mb-6 text-left">
                                    <p className="text-xs font-black uppercase">應付總額 TOTAL</p>
                                    <p className="text-3xl font-black">$ {selectedReceipt.totalAmount}</p>
                                </div>
                                {selectedReceipt.note && <div className="bg-white p-3 rounded-xl border-2 border-dashed border-[#D5BDAF]/30 text-xs italic font-bold text-center text-gray-500">"{selectedReceipt.note}"</div>}
                            </div>
                        </div>
                        <div className="fixed bottom-0 left-0 right-0 bg-white border-t p-4 z-50">
                            <button onClick={() => onCopyLine(selectedReceipt)} className="w-full bg-[#8E7D6F] text-white p-5 rounded-2xl font-black text-xl shadow-xl flex items-center justify-center gap-3 active:scale-95 transition-all">
                                <Icon name="send" /> 一鍵複製明細並回清單
                            </button>
                        </div>
                    </div>
                );
            }

            // --- 視圖：主要分頁介面 ---
            return (
                <div className="min-h-screen pb-24 text-left font-black">
                    <nav className="fixed bottom-0 left-0 right-0 z-40 bg-[#D5BDAF] p-3 flex justify-around shadow-lg rounded-t-[2rem]">
                        {[
                            { id: 'order', label: '下單', icon: 'plus' },
                            { id: 'calc', label: '麵糰', icon: 'chef' },
                            { id: 'list', label: '清單', icon: 'list' },
                        ].map(t => (
                            <button key={t.id} onClick={() => {setActiveTab(t.id); setEditingId(null);}} className={`flex flex-col items-center gap-1 p-2 rounded-2xl transition-all ${activeTab === t.id ? 'bg-[#FDF8F5] text-[#8E7D6F] scale-110 shadow-inner' : 'text-white opacity-70'}`}>
                                <Icon name={t.icon} size={20} /><span className="text-[10px] font-bold">{t.label}</span>
                            </button>
                        ))}
                    </nav>

                    <main className="p-4 max-w-lg mx-auto space-y-6 pt-4">
                        {activeTab === 'order' && (
                            <div className="space-y-4 animate-fade">
                                <div className="bg-white rounded-[2rem] p-6 shadow-sm border border-[#E3D5CA] space-y-4">
                                    <h2 className="text-lg font-black text-[#8E7D6F] flex items-center gap-2"><Icon name="user" size={18} /> {editingId ? '正在修改訂單' : '客戶基本資料'}</h2>
                                    <input type="text" placeholder="客戶姓名" className="w-full bg-[#F9F6F4] rounded-xl p-3 outline-none font-bold" value={currentOrder.name} onChange={e => setCurrentOrder({...currentOrder, name: e.target.value})} />
                                    <input type="tel" placeholder="電話號碼" className="w-full bg-[#F9F6F4] rounded-xl p-3 outline-none font-bold" value={currentOrder.phone} onChange={e => setCurrentOrder({...currentOrder, phone: e.target.value})} />
                                    <div className="grid grid-cols-2 gap-2 text-[11px]">
                                        <div className="flex bg-[#F9F6F4] rounded-xl p-1">
                                            {['自取','外送','Panda'].map(m => (
                                                <button key={m} onClick={()=>setCurrentOrder({...currentOrder, deliveryMethod: m})} className={`flex-1 py-1.5 rounded-lg ${currentOrder.deliveryMethod === m ?'bg-[#8E7D6F] text-white shadow-sm':'text-[#8E7D6F]'}`}>{m}</button>
                                            ))}
                                        </div>
                                        <div className="flex bg-[#F9F6F4] rounded-xl p-1">
                                            {['店內','Line','匯款'].map(m => (
                                                <button key={m} onClick={()=>setCurrentOrder({...currentOrder, paymentMethod: m})} className={`flex-1 py-1.5 rounded-lg ${currentOrder.paymentMethod === m ?'bg-[#8E7D6F] text-white shadow-sm':'text-[#8E7D6F]'}`}>{m}</button>
                                            ))}
                                        </div>
                                    </div>
                                    <input type="text" placeholder="配送詳細地址 (自取可不填)" className="w-full bg-[#F9F6F4] rounded-xl p-3 outline-none text-sm" value={currentOrder.address} onChange={e => setCurrentOrder({...currentOrder, address: e.target.value})} />
                                    <div className="grid grid-cols-2 gap-3">
                                        <input type="text" placeholder="統編" className="bg-[#F9F6F4] rounded-xl p-3 outline-none text-sm" value={currentOrder.taxId} onChange={e => setCurrentOrder({...currentOrder, taxId: e.target.value})} />
                                        <input type="text" placeholder="學校/單位" className="bg-[#F9F6F4] rounded-xl p-3 outline-none text-sm" value={currentOrder.schoolCompany} onChange={e => setCurrentOrder({...currentOrder, schoolCompany: e.target.value})} />
                                    </div>
                                    <input type="datetime-local" className="w-full bg-[#F9F6F4] rounded-xl p-3 outline-none text-[#8E7D6F]" value={currentOrder.deliveryTime} onChange={e => setCurrentOrder({...currentOrder, deliveryTime: e.target.value})} />
                                </div>

                                <div className="bg-white rounded-[2rem] p-6 shadow-sm border border-[#E3D5CA] space-y-4">
                                    <h2 className="text-lg font-black text-[#8E7D6F] flex items-center gap-2"><Icon name="cart" /> 產品規格選擇</h2>
                                    <div className="flex gap-2 overflow-x-auto pb-2 no-scrollbar text-[10px]">
                                        {Object.keys(productCategories).map(cat => (
                                            <button key={cat} onClick={()=>setOrderCategory(cat)} className={`flex-shrink-0 px-4 py-1.5 rounded-full ${orderCategory===cat?'bg-[#8E7D6F] text-white shadow-md':'bg-[#E3D5CA]'}`}>{cat}</button>
                                        ))}
                                    </div>
                                    <div className="grid grid-cols-2 gap-2 text-left font-bold">
                                        {productCategories[orderCategory].map(opt => (
                                            <button key={opt.id} onClick={() => handleAddItem(opt)} className="p-3 bg-[#F9F6F4] rounded-2xl border border-transparent active:border-[#D5BDAF] transition-all">
                                                <p className="text-xs font-black">{opt.label}</p>
                                                <p className="text-[10px] text-[#8E7D6F] font-black">$ {opt.price}</p>
                                            </button>
                                        ))}
                                    </div>
                                    {currentOrder.items.length > 0 && (
                                        <div className="pt-4 border-t border-dashed space-y-2 font-black">
                                            {currentOrder.items.map(i => (
                                                <div key={i.id} className="flex justify-between items-center bg-[#FDF8F5] p-3 rounded-xl border border-[#E3D5CA]/50 text-xs">
                                                    <span className="flex-1">{i.label}</span>
                                                    <div className="flex items-center gap-2">
                                                        <button onClick={() => setCurrentOrder({...currentOrder, items: currentOrder.items.map(it => it.id === i.id ? {...it, qty: Math.max(0, (it.qty||0)-1)} : it).filter(it => it.qty !== 0)})} className="p-1 active:scale-75 text-[#D5BDAF]"><Icon name="minus" size={14}/></button>
                                                        <input type="number" value={i.qty} onChange={(e) => {const val = e.target.value === '' ? '' : parseInt(e.target.value)||0; setCurrentOrder({...currentOrder, items: currentOrder.items.map(it => it.id === i.id ? {...it, qty: val} : it)})}} className="w-10 bg-white border rounded p-1 text-center outline-none" />
                                                        <button onClick={() => setCurrentOrder({...currentOrder, items: currentOrder.items.map(it => it.id === i.id ? {...it, qty: (it.qty||0)+1} : it)})} className="p-1 active:scale-75 text-[#D5BDAF]"><Icon name="plus" size={14}/></button>
                                                    </div>
                                                </div>
                                            ))}
                                        </div>
                                    )}
                                </div>

                                <div className="bg-white rounded-[2rem] p-6 shadow-sm border border-[#E3D5CA] space-y-4 text-left">
                                    <h2 className="text-lg font-black text-[#8E7D6F] flex items-center gap-2"><Icon name="gift" /> 加購項目與備註</h2>
                                    <div className="grid grid-cols-2 gap-2 text-[10px]">
                                        {Object.entries(extraNames).map(([k, label]) => (
                                            <button key={k} onClick={()=>setCurrentOrder({...currentOrder, extras: {...currentOrder.extras, [k]: !currentOrder.extras[k]}})} className={`p-3 rounded-xl border flex items-center justify-between transition-all ${currentOrder.extras[k]?'bg-[#D5BDAF] text-white':'bg-[#F9F6F4]'}`}><span>{label}</span>{currentOrder.extras[k]&&<Icon name="check" size={14}/>}</button>
                                        ))}
                                    </div>
                                    <textarea placeholder="特別要求備註..." className="w-full bg-[#F9F6F4] rounded-xl p-4 outline-none text-sm min-h-[80px]" value={currentOrder.note} onChange={e=>setCurrentOrder({...currentOrder,note:e.target.value})}></textarea>
                                </div>
                                <button onClick={handleSaveOrder} className="w-full bg-[#8E7D6F] text-white p-5 rounded-[2rem] font-black text-xl shadow-xl active:scale-95 transition-all">{editingId ? '確認修改訂單' : '確認儲存訂單'}</button>
                                {editingId && <button onClick={() => {setEditingId(null); handleSaveOrder();}} className="w-full text-gray-400 font-bold text-sm">取消修改</button>}
                            </div>
                        )}

                        {activeTab === 'calc' && (
                            <div className="bg-white rounded-[2rem] p-8 shadow-sm border border-[#E3D5CA] space-y-6 animate-fade font-black text-left">
                                <h2 className="text-xl font-black text-[#8E7D6F] flex items-center justify-center gap-3"><Icon name="chef" /> 當前揉麵總量需求</h2>
                                <div className="space-y-4">
                                    {doughTotal.map(d => (
                                        <div key={d.name} className="p-6 bg-[#FDF8F5] rounded-3xl border border-[#D5BDAF]/20 flex justify-between items-center shadow-inner">
                                            <p className="text-xl text-[#5E503F]">{d.name}</p>
                                            <div className="text-right">
                                                <p className="text-2xl text-[#8E7D6F]">{d.batches > 0 ? `${d.batches} 團 又 ${d.remainder} 顆` : `1 團 (需求 ${d.total} 顆)`}</p>
                                                <p className="text-[10px] opacity-40 uppercase tracking-widest">揉麵單位 (1團=80顆)</p>
                                            </div>
                                        </div>
                                    ))}
                                    {doughTotal.length === 0 && <div className="py-24 text-center opacity-30 italic text-lg w-full">目前無訂單製作需求</div>}
                                </div>
                            </div>
                        )}

                        {activeTab === 'list' && (
                            <div className="space-y-4 animate-fade text-left font-black">
                                <h2 className="text-lg font-black text-[#8E7D6F] text-center mb-2 font-black uppercase tracking-widest">訂單清單</h2>
                                {orders.map(o => (
                                    <div key={o.id} className="bg-white rounded-[2rem] p-5 border border-[#E3D5CA] shadow-sm relative overflow-hidden group">
                                        <div className="absolute top-0 left-0 w-1.5 h-full bg-[#D5BDAF] opacity-30"></div>
                                        <div className="flex justify-between items-start mb-3">
                                            <div onClick={() => {setSelectedReceipt(o); setView('receipt');}} className="flex-1 cursor-pointer">
                                                <h3 className="text-xl font-black text-[#5E503F] underline decoration-[#D5BDAF]/50 underline-offset-4 decoration-dashed">{o.name || '客戶'}</h3>
                                                <p className="text-lg font-black text-[#8E7D6F] mt-1">$ {o.totalAmount}</p>
                                                <p className="text-[10px] font-bold opacity-30 mt-1 uppercase tracking-widest">{o.deliveryMethod} • {o.paymentMethod}</p>
                                            </div>
                                            <div className="flex flex-col gap-2">
                                                <button onClick={() => setOrders(orders.filter(ord=>ord.id!==o.id))} className="text-red-200 p-2 active:scale-125"><Icon name="trash" size={18}/></button>
                                                {/* 修改訂單按鈕 */}
                                                <button onClick={() => handleEditClick(o)} className="text-[#D5BDAF] p-2 active:scale-125"><Icon name="edit" size={18}/></button>
                                            </div>
                                        </div>
                                        
                                        <div className="bg-[#FDF8F5] p-3 rounded-2xl border border-[#D5BDAF]/20 shadow-inner">
                                            <p className="text-[9px] font-black text-[#8E7D6F] uppercase mb-1 flex items-center gap-1"><Icon name="target" size={10}/> 麵糰計量參考 (1團=80顆)</p>
                                            {getDoughDemand(o.items).map(d => (
                                                <div key={d.name} className="flex justify-between text-[11px] py-0.5 border-b border-white last:border-0">
                                                    <span>{d.name}</span>
                                                    <span className="text-[#8E7D6F]">{d.batches > 0 ? `${d.batches} 團 又 ${d.remainder} 顆` : `1 團 (僅需 ${d.remainder} 顆)`}</span>
                                                </div>
                                            ))}
                                        </div>
                                    </div>
                                ))}
                                {orders.length === 0 && <div className="py-24 text-center opacity-30 italic font-black">目前清單是空的</div>}
                            </div>
                        )}
                    </main>
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
